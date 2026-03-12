
# Batch Concept and Usage in the legacy SLC Application

This document describes how **Batch** is defined, stored, and used in the SLC (Scale Labeling Station) application. The concept spans database tables, MII/webservice integration, Produce-to-Goal flows, serial creation, label printing, and maintenance (clear/purge).

---

## 1. Overview of “Batch” in SLC

In this codebase, **batch** is used in several related but distinct ways:

1. **BatchOrder table** — Persisted batch records keyed by batch date and optionally order/product, used to drive “Produce to Goal” by batch (date + product) and to link goals to a **PDN Batch ID**.
2. **PDN Batch ID (PDNBatchID)** — An **int64** identifier that ties a **Serial** (printed case) or a **Goal** to a batch. It is stored on **Serial.PDNBatchID** and passed through the global **gPDNBatchID** when producing to a batch goal.
3. **TT-Batch (temp table)** — In-memory batch data from MII/webservices (ProcessOrder, BatchNumber, PackDate, KillDate, SellByDate, Line, Material, etc.) used in product-process flows (e.g. combo ID, container ID, line + prodcode) to select a batch and populate modify/serial data.
4. **Batch ID (4-digit, 1000–9399 or 9878)** — A separate, legacy “Batch ID” captured in **print/batch-id.w**, stored in site config **SerialNVPs** as the name-value pair **BatchID**, and rendered on labels via the **BatchID** token in **lib/printlabel.p**.
5. **case-batch-serial.p** — A product-process routine that prompts for “number of cases required” and creates that many serials/labels in a loop (batch of labels per user request).

The sections below detail each of these and how they connect.

---

## 2. Database Schema

### 2.1 BatchOrder Table

**Definition**: `data/BatchOrder.df`

| Field      | Type    | Description |
|-----------|---------|-------------|
| BatchID   | int64   | Unique batch identifier (from host/PDN). |
| BatchDate | date    | Batch date; part of logical key for batch-by-date flows. |
| OrdNum    | character (10) | Order number when batch is order-linked. |
| OrdRefNum | int64   | Order reference number. |
| OrdRefLine| int64   | Order reference line. |
| ProdCode  | integer | Product code. |

**Index**: `BatchDate` (primary, ascending).

**Purpose**:
- Stores one row per batch known to the SLC. Rows are created when the **host** sends a goal download (**msg-goal-download.p**) and a goal’s **Description** matches batch format and has no order (see §4.1).
- **Goals** in “Produce to Goal” can be selected **by batch**: user picks a **batch date** and **product**; the UI loads goals that are linked to **BatchOrder** by (BatchDate, ProdCode) or by (OrdNum, OrdRefLine). The chosen goal’s **PDNBatchID** is set from **BatchOrder.BatchID** and stored in **gPDNBatchID** so that every serial produced to that goal gets **Serial.PDNBatchID = gPDNBatchID**.

**Maintenance**: **lib/clearlogs.p** purges old **BatchOrder** rows. It uses site config **"BatchOrder Clear Days"** (default 7). For each **BatchOrder** with **BatchDate < TODAY - v-BatchOrder-Clear-Days**, it checks that no **Goals** reference that batch (by OrdNum/OrdRefLine or ProdCode/TgtCompDate/BatchDate), then **DELETE BatchOrder**. So batch records are retained only for the configured window and only if not still referenced by goals.

### 2.2 Serial.PDNBatchID

**Definition**: `data/Serial-PDNBatchID.df` adds to the **Serial** table:

| Field     | Type  | Description |
|----------|-------|-------------|
| PDNBatchID | int64 | Production/batch identifier; links this serial to a batch (e.g. from host or MII). |

**Usage**: When a serial is created in **lib/createserial.p**, the code assigns:

```progress
pb-Serial.PDNBatchID = gPDNBatchID
```

So every case (serial) produced while **gPDNBatchID** is set (e.g. after selecting a batch goal) is stamped with that batch ID. When the user is not producing to a batch goal, **gPDNBatchID** is 0 (e.g. reset in main menu and in fresh-goal flow).

---

## 3. TT-Batch Temp Table (MII / In-Process Batches)

**Definition**: `print/costing/TT-Batch.i`

```progress
DEF TEMP-TABLE TT-Batch NO-UNDO
    FIELD ProcessOrder   AS  c
    FIELD BatchNumber    AS  c
    FIELD OrderType      AS  c    /* "MFG" or "Pack" */
    FIELD PackDate       AS  DATE
    FIELD KillDate       AS  DATE
    FIELD SellByDate     AS  DATE
    FIELD SubResource    AS  c
    FIELD MessageType   AS  c
    FIELD LINE          AS  c
    FIELD Material       AS  c
    FIELD MessageFlag    AS  c
    FIELD MessageID     AS  c
    FIELD MessageText   AS  c
    INDEX Order AS PRIMARY UNIQUE ProcessOrder.
```

**Purpose**: Holds batch data returned from MII (or built from webservice response). Used in:
- **bridge-get-batches-for-container-id.p** — Input: container ID; output: TT-Batch table (batches for that container).
- **bridge-get-batches-for-line-prodcode.p** — Input: line, prod code; output: TT-Batch table (batches for that line/product).
- **bridge-build-batches-from-websvc-tt.p** — Parses MII SOAP/XML response (e.g. ProcessOrder, KillDate, PackDate, SellByDate, Material, Line) and populates TT-Batch.

**Flow (e.g. op-set-combo-id.w)**:
- User enters a “Combo ID” (container ID). Program runs **bridge-get-batches-for-container-id.p** with that ID; MII returns batch data into **TT-Batch**.
- First TT-Batch row is used to set **ProcessOrder**, **PackDate**, **KillDate**, **SellByDate**, **Material**, **Line**, **SubResource**, etc., and to set **b-tt-Modify.Lot = TT-Batch.BatchNumber**. These values are then used for serial creation and for **set-MII-fields.p** (which pushes batch/product/lot/date data into site config or NVP for labels).
- **LastsForCycle-Batch** site config controls whether batch selection lasts for the “Serial” (prompt every time and store batch data for serial) or for a longer cycle (e.g. “MainMenu”) where batch data is reused from TT-Site-Value without calling MII again.

So **TT-Batch** is the in-memory representation of MII batches for the current product/container/line; **BatchOrder** is the persisted batch list used for goal selection and **Serial.PDNBatchID**.

---

## 4. Batch and Goals (Produce to Goal)

### 4.1 How Batch Goals Arrive: msg-goal-download.p

When the host sends a **Goal Download** message:

- Goals are read from the message body into temp table **tt-Goals** and written to **Goals**.
- For each goal, if **Description** matches **"Batch:*"** and **OrdNum** is null or blank, the code treats it as a **batch goal**:
  - It parses **v-BatchID** from **Description** (e.g. first part of the second entry when split by ":", then by "-").
  - It looks up **BatchOrder** by **BatchID**, **BatchDate** (= **TgtCompDate**), and **ProdCode**.
  - If no **BatchOrder** row exists, it **CREATE**s one and sets **BatchID**, **BatchDate**, **ProdCode** (OrdNum, OrdRefNum, OrdRefLine are not set in this path).

So batch goals from the server both create and rely on **BatchOrder** rows. Those rows are then used on the SLC to list “batch” goals by date and product.

### 4.2 Site Config: gGoalsSelectBatch

- **gGoalsSelectBatch** (from **lib/GlobalAssigns.i**, site session parm **"Goal Select By Date"**) controls whether the main menu offers **batch** goal selection.
- In **sobjects/s-mainmenu.w**:
  - When opening “Produce to Goal”, **gPDNBatchID** is set to 0.
  - If **gGoalsSelectBatch** is true, the app runs **goals/w-select-batch-date.w** so the user picks a **date** first, then product (and optionally goal list).
  - After the goal selection flow, **gPDNBatchID** is again set to 0 when leaving (e.g. when not choosing a goal).

### 4.3 Produce to Goal by Batch: Date → Product → Goals

1. **goals/w-select-batch-date.w**  
   User enters or selects a **batch date**. Then the app runs **goals/w-select-batch-product.w** with that date.

2. **goals/w-select-batch-product.w**  
   User selects a **product code**. Then the app runs **goals/w-select-batch-goals.w(INPUT ip-BatchDate, INPUT ip-ProdCode)**.

3. **goals/w-select-batch-goals.w**  
   - **BuildGoalList** (or equivalent) loads goals that belong to that date and product by scanning **BatchOrder** for the given **BatchDate** and **ProdCode**.
   - For each **BatchOrder** row, it finds **Goals** that match either:
     - **Goals.ProdCode = BatchOrder.ProdCode** and **Goals.TgtCompDate = BatchOrder.BatchDate** (batch-by-date/product), or
     - **Goals.OrdNum = BatchOrder.OrdNum** and **Goals.OrdRefLine = BatchOrder.OrdRefLine** (order-linked).
   - Only goals that are not canceled and not complete (or complete but within tolerance) are included.
   - For each goal, a **tt-Goals** row is created with **tt-Goals.PDNBatchID = BatchOrder.BatchID**.
   - Goal buttons are built with a name that includes the goal and batch ID, e.g. **'GOAL'-STRING(tt-Goals.GoalID)-STRING(tt-Goals.PDNBatchID)**.

4. When the user **chooses a goal** from that list:
   - **vPDNBatchID** is parsed from the button name (e.g. third entry when split by "-").
   - **gPDNBatchID = vPDNBatchID** is set.
   - **gGoalID** and **gGoalsRowid** are set from the selected goal, and the weigh window is run with **StartNewProduct** so production runs against that goal. Every serial created from then on gets **Serial.PDNBatchID = gPDNBatchID** until the user exits or clears the goal.

### 4.4 Fresh Goals vs Batch Goals

- **w-select-fresh-goals.w** is used when **gGoalsSelectBatch** is false (or for “fresh” path). There, **tt-Goals.PDNBatchID** is set to 0 and **gPDNBatchID = 0** when leaving, so serials are not tied to a batch ID from the batch-goal flow.
- **weigh-enableafteroutput.p** checks **Goal Select By Date** (batch mode); if true (**v-BatchGoals**), it can alter behavior (e.g. when to refresh or what to show) after a label is printed.

---

## 5. Serial Creation and gPDNBatchID

- **lib/createserial.p** assigns **pb-Serial.PDNBatchID = gPDNBatchID** when creating a new serial. No other logic in that file overwrites it; so whenever the user is producing to a batch goal, **gPDNBatchID** is set before the weigh/print flow and every new serial gets that batch ID.
- **gPDNBatchID** is declared in **lib/gProcessingVars.i** as a global shared **int64**. It is set:
  - To **0** in main menu when entering/leaving Produce to Goal and in fresh-goal flow.
  - To **vPDNBatchID** in **w-select-batch-goals.w** when the user selects a batch goal (see above).

---

## 6. Batch ID (4-Digit) and Labels — batch-id.w and SerialNVPs

This is a **different** concept from **PDNBatchID**:

- **print/batch-id.w** prompts the user for **fi-BatchID** (4 characters, numeric).
- **SetBatchID** validates: integer in range 1000–9399 or 9878 (test batch), no decimal point, length 4.
- The value is stored in site config **SerialNVPs** as the name-value pair **BatchID** (via **f-NVP-set-value** and **f-Set-Site-Value-TT**). So it’s a **session/site-level** “Batch ID” used for labeling, not a row in **BatchOrder**.
- **lib/printlabel.p** supports a label token **"BatchID"**: it reads **BatchID** from **pb-CaseLabelSource.UserBarString** (NVP) and uses it as replacement text. So when labels are printed, the current “Batch ID” from SerialNVPs can be embedded on the label.
- **batch-id.w** is typically used as a **product process** step (e.g. before case weight) so the operator sets the 4-digit batch ID once per run; that value is then available for label tokens and NVP storage.

---

## 7. case-batch-serial.p (Batch of Serials per Request)

**File**: `print/case-batch-serial.p`

- This is a **product process output routine** (like case-print, but for creating multiple serials in one go).
- It prompts: **“Enter Number of Cases Required”** and updates **vBatchCasesRequired**.
- Then it loops **vBatchCaseCtr = 1 to vBatchCasesRequired** and, for each iteration, calls **CreateSerial** in **gh_CreateSerial** with the same weight and modify data, and sends the label to the device. So one user action generates **N** serials and **N** labels (same product/modify, same weight).
- **set-labels.p** and **set-label-wgt.p** reference **case-batch-serial.p** as the routine that drives this “batch of cases” behavior. This is “batch” in the sense of “batch of labels/serials,” not the **BatchOrder** or **PDNBatchID** concept.

---

## 8. MII Webservice Bridges (TT-Batch Population)

| Program | Input | Output | Purpose |
|--------|-------|--------|---------|
| **bridge-get-batches-for-container-id.p** | Container ID | TT-Batch table, success, message | Get batches for a container (e.g. combo ID) from MII. |
| **bridge-get-batches-for-line-prodcode.p** | Line, ProdCode | TT-Batch table, success, message | Get batches for a line and product from MII. |
| **bridge-build-batches-from-websvc-tt.p** | TT-Element (parsed XML), NVP, Debug | TT-Batch table | Build TT-Batch from MII SOAP/XML response (ProcessOrder, PackDate, KillDate, SellByDate, Material, Line, etc.). |

These are used by **op-set-combo-id.w** (container ID → batches) and by **op-fg-wip-pack-set-order.w** (line/product → batches; “FG WIP Pack” process mode). The resulting **TT-Batch** drives ProcessOrder, Lot (BatchNumber), dates, and line/material for the current run and for **set-MII-fields.p**.

---

## 9. Product Process and Batch

- **op-set-combo-id.w**: Prompts for “Combo ID”; calls **bridge-get-batches-for-container-id.p**; fills TT-Batch and sets modify/serial-related data (lot, process order, line, etc.). **LastsForCycle-Batch** (“Serial” vs “MainMenu”) controls whether batch selection is per serial or lasts until main menu.
- **op-fg-wip-pack-set-order.w**: Uses TT-Batch to list process orders; user picks one; batch data (BatchNumber, dates, line, etc.) is applied. Comments reference **op-select-batch-browse.w** (which may be removed or renamed); the flow “must be before” that browse in the process sequence when both are used.
- **op-fg-wip-pack-set-date.w** and **op-fg-wip-pack-set-batch.w**: Read-only dates/batch set by the batch selection step (e.g. op-select-batch-browse or combo-id); they do not change batch here.
- **print/set-MII-fields.p**: Takes **TT-Batch** and **ProcessOrder** as input; pushes batch number, pack date, kill date, sell-by date, etc., into site config (e.g. for label or downstream use). It is used with **op-select-batch-browse.w** in comments; the same TT-Batch is passed through the process.

---

## 10. Clearlogs and BatchOrder Purge

In **lib/clearlogs.p**:

- **BatchOrder Clear Days** is read from site config (default 7). If the option does not exist, it is created in ConfigOptionMaster and SiteConfigOption with default 7.
- For each **BatchOrder** where **BatchDate < TODAY - v-BatchOrder-Clear-Days**:
  - The code checks that no **Goals** row references this batch (either by OrdNum/OrdRefLine matching BatchOrder, or by ProdCode and TgtCompDate and BatchDate matching).
  - If no such goal exists, **DELETE BatchOrder**.

So **BatchOrder** is purged by age and only when no goal still references it.

---

## 11. Summary Table

| Concept | Where it lives | Main use |
|--------|----------------|----------|
| **BatchOrder** | DB table (BatchOrder.df) | Persisted batch list; created from batch goals (msg-goal-download); drives “Produce to Goal by batch” and BatchOrder.BatchID → gPDNBatchID. |
| **Serial.PDNBatchID** | Serial table (Serial-PDNBatchID.df) | Links each printed serial to a batch (gPDNBatchID at create time). |
| **gPDNBatchID** | Global (gProcessingVars.i) | Set when user selects a batch goal; copied to every new serial until flow resets. |
| **TT-Batch** | Temp table (TT-Batch.i) | MII batch data (ProcessOrder, BatchNumber, dates, Line, Material); used in combo-id and FG WIP Pack flows and set-MII-fields. |
| **BatchID (4-digit)** | SerialNVPs (site config) | Operator-entered batch ID (1000–9399 or 9878); stored as NVP “BatchID”; printed on labels via BatchID token. |
| **case-batch-serial.p** | Print routine | Asks “how many cases” and creates that many serials/labels in one go (batch of cases). |

---

## 12. Redevelopment Notes

- Replicate **BatchOrder** and **Serial.PDNBatchID** in the new schema so batch goals and serial–batch linkage are preserved.
- Keep **gPDNBatchID** (or equivalent session variable) and set it when the user selects a batch goal; pass it into serial creation.
- Replicate goal-download logic that creates **BatchOrder** when **Description** matches **"Batch:*"** and OrdNum is empty, and parses BatchID from Description.
- Replicate “Produce to Goal by batch”: date → product → list goals from BatchOrder + Goals; on goal selection set PDNBatchID and use it for every new serial until exit.
- **TT-Batch** and MII bridges can be reimplemented as a “batch service” that returns process order, batch number, dates, line, material; the UI (combo ID or line/product selection) and set-MII-fields behavior should mirror the current flow.
- **BatchOrder Clear Days** and clearlogs logic should be preserved so old batch rows are purged only when no goal references them.
- The 4-digit **BatchID** (batch-id.w + SerialNVPs + BatchID token) is independent: keep NVP storage and label token **BatchID** if labels must show that value.
- **case-batch-serial** is “N serials per request”; reimplement as one API/action that creates N serials with the same payload and prints N labels.
