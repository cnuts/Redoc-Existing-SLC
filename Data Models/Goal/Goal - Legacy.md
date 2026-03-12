# Goal Concept and Usage in the legacy SLC Application

This document describes the **Goal** concept in full: what a goal represents, how it is stored, how it is selected and used during production, how progress is updated, and how goals are purged or synced with the host.

---

## 1. Overview

A **Goal** is a production target that the SLC (Scale Labeling Station) works toward when running in **Produce to Goal** (also referred to as Produce to WO) mode. Each goal has:

- A **target** (count, net weight, or label weight) and optional **tolerance**.
- A **product** (ProdCode), optional **order** (OrdNum, OrdRefLine), and **target completion date/time** (TgtCompDate, TgtCompTime).
- **Status** (e.g. Ready, In Process, Complete, Canceled) and **totals** (current and all-scale count/weights).
- Optional packaging, label, and customer data used when printing labels for that goal.

The operator selects a goal from the main menu (by batch date/product, by fresh goals list, by goal directly, or by order). The weigh window then runs with that goal: every serial (case) produced is tied to the goal via **Serial.GoalID** and **ItemSerial.GoalID**, and the goal’s totals and status are updated when serials are added or deleted (**lib/updatetotals.p**, **lib/updatetotals2.p**). Goals are created or updated from the **host** via **Goal Download** messages and can be purged locally by age (**Goals-Purge-Days**). The **Progress Host** runs **goals-setcount-complete.p** to sync all-scale counts and status with the central system.

---

## 2. Database Schema: Goals Table

**Source**: `slc.df` (table **Goals**).

### 2.1 Identity and Product

| Field     | Type     | Description |
|----------|----------|-------------|
| GoalID   | integer  | Primary key. Unique goal identifier. |
| ProdCode | integer  | Product code for this goal. |
| LimitBy  | character | How the target is measured: **Count**, **NetWgt**, or **LabelWgt**. |
| TargetLevel | decimal | Target value (count or weight). |
| Tolerance | decimal | Optional tolerance; goal can be considered achieved at TargetLevel + Tolerance. |

### 2.2 Order and Schedule

| Field       | Type     | Description |
|------------|----------|-------------|
| OrdNum     | character | Customer/order number (from Inven). |
| OrdRefLine | integer  | Order line/reference. |
| TgtCompDate | date    | Target completion date. |
| TgtCompTime | character | Target completion time (e.g. HH:MM). |
| Priority   | integer  | Priority ordering (used with GoalStatus, ProdCode in index). |

### 2.3 Status and Totals

| Field                | Type    | Description |
|---------------------|---------|-------------|
| GoalStatus          | character | See §3. Values: `' '` (Ready), `InProcess`, `Complete`, `CANCELED`, etc. |
| AllScaleTotalCount  | integer | Total count across all scales (from host or SLC updates). |
| AllScaleTotalNetWgt | decimal | Total net weight, all scales. |
| AllScaleTotalLabelWgt | decimal | Total label weight, all scales. |
| CurrentCount        | integer | This scale’s count (legacy/local). |
| CurrentLabelWgt     | decimal | This scale’s label weight. |
| CurrentNetWgt       | decimal | This scale’s net weight. |
| SentToServer        | logical | Whether completion has been sent to server. |
| SentToWPL           | logical | Whether sent to WPL. |
| DateStatusChange    | date   | Date status last changed (used for purge). |

### 2.4 Display and NVP Data

| Field        | Type     | Description |
|-------------|----------|-------------|
| Description | character | Short description (e.g. "Batch:1234-..."). |
| CustomerName | character | Customer name. |
| CustomerNumber | character | End customer ID. |
| PkgDesc3Size | character | Name-value pairs (CHR(2)-separated); includes **PrintedCount**, **PrintedWgt**, **Printed1**, **Printed2** and other keys. Preserved on download; SLC updates printed counts/weights here. |

### 2.5 Packaging, Label, and Product Overrides

Goals can override product-level settings for labels and packaging. Examples (non-exhaustive):

- **WgtRoundOrTruncate**, **WarningLevel**, **ScalesInGoal**
- **PkgDesc1**, **PkgDesc2**, **PkgDesc3**, **PkgDesc1Size**, **PkgDesc2Size**
- **PrintLabel**, **LabelFile**, **PkgLabelFile**, **BarCodeType**, **BarCodePositionID**, **PrintPackDate**, **PkgDateFormat**
- **PackDateOffset**, **SellByDate**, **PrintPkgSellBy**, **PrintCaseSellBy**, **PrintExtraMessage**
- **UnitPrice**, **UnitTare**, **BoxTare**, **TrayMinWgt**, **TrayMaxWgt**, **TrayCount**, **TraySize**, **TrayLineSpeed**
- **HeadHeight**, **LabelPosition**, **Grade**, **MfgID**
- **DateAI**, **DateAIDate** (used in serial creation for date logic)
- **WO_Number**, **WOLine**, **AddMsg1**–**AddMsg3**, **ProdDesc1**, **ProdDesc2**, **LabelFormat**, **PkgMaxLabelWgt**
- **MarkDownMode**, **MarkDownPrice**, **MarkDownMethod**, **SalesMode**, **CasesPerPallet**, **PalletCluster**, **CoolerProductCode**, **PalletizerLane**, **WetTareWgt**, **WetTarePct**, **WetTareType**, **TareType**, **CustomerCode**, **DateCode**, **ExactCasesPerPalletRequired**, **ExactItemsPerCaseRequired**, and others.

### 2.6 Indexes

- **GoalID** (primary, ascending).
- **GoalStatus** (GoalStatus, GoalID).
- **Priority** (GoalStatus, Priority, ProdCode, GoalID).
- **ProdCode** (ProdCode).
- **TgtDateTimeGoal** (TgtCompDate, TgtCompTime, GoalID).

---

## 3. Goal Status Values

**Defined in** `lib/prepr.i`:

| Symbol         | Value       | Meaning |
|----------------|------------|---------|
| GoalReady      | `' '` (space) | Ready to be selected. |
| GoalInProcess  | `InProcess`  | Currently being produced. |
| GoalComplete   | `Complete`   | Target (or target + tolerance) reached. |

Other statuses appear in the code (e.g. **CANCELED**, **GoalCanceledAtCTS**). Goals with status **CANCELED** (or canceled at CTS) are excluded from host sync in **goals-setcount-complete.p** and from certain selection lists.

**Status transitions** (see §8):

- When **AllScaleTotalCount** (or NetWgt/LabelWgt per **LimitBy**) reaches **TargetLevel** → **GoalStatus = 'Complete'**, **SentToServer = no**.
- When a goal was **Complete** and totals later fall at or below **TargetLevel** (e.g. serials deleted) → **GoalStatus = 'InProcess'**, **SentToServer = no**.

---

## 4. Host Goal Download (msg-goal-download.p)

**File**: `host/msg-goal-download.p`

- **Input**: **ip-MsgBody** (LONGCHAR) — JSON body containing goal data.
- **Output**: **op-Successful**, **op-MsgString** (comma-separated list of GoalIDs processed).

**Behavior**:

1. Message body is read into temp table **tt-Goals** (like Goals, with extra **RescheduleFlag**).
2. For each **tt-Goals** row:
   - **FIND Goals** by **GoalID** (exclusive-lock, no-wait). If not found, **CREATE** Goals; else update.
   - **BUFFER-COPY** from tt-Goals to Goals **EXCEPT** PkgDesc3Size, CurrentCount, CurrentLabelWgt, CurrentNetWgt (so server does not overwrite local progress).
   - **Goals.DateStatusChange = TODAY**.
   - **Goals.AllScaleTotalCount**, **AllScaleTotalNetWgt**, **AllScaleTotalLabelWgt** are set from **tt-Goals.CurrentCount/CurrentNetWgt/CurrentLabelWgt** (server’s view of totals).
   - **PkgDesc3Size**: each entry in tt-Goals.PkgDesc3Size (CHR(2)-separated, "name=value") is merged into Goals.PkgDesc3Size via **f-NVP-set-value**, except keys **PrintedCount**, **PrintedWgt**, **Printed1**, **Printed2**, **RescheduleFlag** (SLC maintains these).
   - If **tt-Goals.RescheduleFlag = "Yes"**: **CurrentCount**, **CurrentLabelWgt**, **CurrentNetWgt** set to 0; **PkgDesc3Size** PrintedCount/PrintedWgt set to "0".
3. If **Description** matches **"Batch:*"** and **OrdNum** is null or blank, the program parses **BatchID** from Description and ensures a **BatchOrder** row exists (see Batch concept doc).

Goals on the SLC are thus created/updated from the host; local printed counts and weights are kept in **PkgDesc3Size** and in **Current*/AllScaleTotal*** as the SLC and host update them.

---

## 5. Goal Selection: Main Menu and Flows

**File**: `sobjects/s-mainmenu.w` — **Produce to Goal** button (e.g. **imgGoal**).

- **gProduceToWo = yes**, **gPDNBatchID = 0**.
- Which dialog runs depends on session globals (from **GlobalAssigns.i**):
  - **gGoalsSelectBatch** → **goals/w-select-batch-date.w** (then batch product → batch goals).
  - Else **gGoalsSelectGoal** → **goals/w-select-goal-btns.w** (select goal directly).
  - Else **gGoalsSelectOrder** → **goals/w-select-order-btns.w** (select by order).
  - Else **goals/select-order-browse.w** (order browse).

So **Goal** selection can be:

1. **By batch**: Date → Product → List of goals (from BatchOrder + Goals); user picks one; **gPDNBatchID** and **gGoalID**/gGoalsRowid set (see Batch doc).
2. **By goal**: User picks from a list of goals (e.g. **w-select-goal-btns.w** or **w-select-fresh-goals.w**).
3. **By order**: User picks order (and optionally line) then goal.

After a goal is chosen, the code typically runs **CheckTargetTime** (see §10); if OK, sets **gGoalID = Goals.GoalID**, **gGoalsRowid = rowid(Goals)**, and runs the weigh window (**print/weigh.w**) via **StartNewProduct(Goals.GoalID, Goals.ProdCode, ...)**.

---

## 6. Produce to Goal: gGoalID, gGoalsRowid, StartNewProduct

**Globals** (`lib/gProcessingVars.i`):

- **gGoalID**: **Goals.GoalID** for the current production run (or ? when not producing to a goal).
- **gGoalsRowid**: **rowid(Goals)** so the app can re-find the goal without holding a lock.

**print/weigh.w**:

- When opened from **Produce to Goal**, the caller sets **gGoalID** and **gGoalsRowid** and runs **StartNewProduct(gGoalID, gProductCode, handle)**. If **gProduceToWo** is false (e.g. Print Label only), **gGoalID** is set to ? and goal is released.
- **StartNewProduct** finds **Goals** by **gGoalID**, sets **gGoalsRowid = rowid(Goals)**. If **Goals.GoalStatus = ''**, it runs **UpdateGoalStatus(GoalInProcess)**. The weigh UI then uses **Goals** (via gGoalsRowid) for product, label, and totals display.
- **GetGoalProdTotal** (in weigh.w) returns production total, count, target+tolerance, and tolerance remaining based on **Goals.LimitBy** and AllScale/Current totals (see §9).
- When the operator finishes or switches product, **gGoalID** and **gGoalsRowid** can be cleared (e.g. **gGoalID = ?**, **gGoalsRowid = ?**).

So the **current goal** for the weigh session is identified by **gGoalID** and **gGoalsRowid**; all serials created during that session can be tied to that goal.

---

## 7. Serial and ItemSerial: GoalID

**Schema**:

- **Serial.GoalID** (integer): Set when the serial is created for a goal.
- **ItemSerial.GoalID** (integer): Same idea for item-level serials.

**lib/createserial.p**:

- **FIND Goals** where **rowid(Goals) = gGoalsRowid** (no-lock). If **AVAILABLE(Goals)**, **pb-serial.GoalID = Goals.GoalID**; else **pb-serial.GoalID = ?** (or equivalent). So every serial created while a goal is active gets that **GoalID**. **Goals** also supply **DateAI**, **DateAIDate**, **SellByDate** (and offsets) for the serial when available.

---

## 8. Counter-Goal CrossRef

**lib/createserial.p**:

- When **Goals** is available (produce to goal), the code finds or creates **CrossRef** where **Application = "Counter-Goal"** and **ID = STRING(Goals.GoalID)**. **Descr** holds a string counter (incremented on add, decremented on delete) for that goal. This is used for serial numbering or sequencing within the goal (e.g. case number within goal). When a goal is purged (see §12), the corresponding **Counter-CrossRef** row is deleted.

---

## 9. GetGoalProdTotal (weigh.w)

**Procedure GetGoalProdTotal** (outputs: production total, production count, target+tolerance, tolerance remaining):

- **opPdnTargetPlusTolerance = Goals.TargetLevel + Goals.Tolerance**.
- **opProductionCount** = max(Goals.AllScaleTotalCount, Goals.CurrentCount).
- **opProductionTotal** depends on **Goals.LimitBy**:
  - **Count** → max(AllScaleTotalCount, CurrentCount).
  - **NetWgt** → max(AllScaleTotalNetWgt, CurrentNetWgt).
  - **LabelWgt** → max(AllScaleTotalLabelWgt, CurrentLabelWgt).
- **opToleranceRemaining = opPdnTargetPlusTolerance - opProductionTotal**.

The weigh window uses these to show progress toward the goal (e.g. “Reached Target” / “Reached Tolerance” or remaining amount).

---

## 10. Updating Goals When Serials Are Added or Deleted

### 10.1 Entry Points

- **lib/updatetotals.p** (procedure **UpdateTotals**): Updates **Totals** by shift/scale/date/product; if **pb-serial.GoalID <> ?**, runs **UpdateGoal-Trans** in the same transaction.
- **lib/updatetotals2.p**: Creates/updates **Totals** and, if **pb-serial.GoalID <> ?**, updates the **Goals** record: **PkgDesc3Size** (PrintedCount, PrintedWgt, Printed1, Printed2), **CurrentCount**, **CurrentLabelWgt**, **CurrentNetWgt**, **AllScaleTotalCount**, **AllScaleTotalNetWgt**, **AllScaleTotalLabelWgt**, and **GoalStatus** based on **LimitBy** and **TargetLevel**/**Tolerance**.

So both paths can update goal totals and status when a serial is added (Add) or removed (Delete); **updatetotals2.p** is the one that explicitly maintains **PkgDesc3Size** and AllScale totals and status transitions.

### 10.2 Factor and Status Logic (updatetotals.p / updatetotals2.p)

- **vAddDeleteFactor**: +1 for Add, -1 for Delete.
- **Goals** is found by **GoalID** (exclusive-lock with retry/timeout in updatetotals.p; similar in updatetotals2.p).
- **PkgDesc3Size**: **PrintedCount**, **PrintedWgt**, **Printed1**, **Printed2** are read with **f-NVP-get-value**, adjusted by **vAddDeleteFactor**, and written back with **f-NVP-set-value**.
- **CurrentCount**, **CurrentLabelWgt**, **CurrentNetWgt** and **AllScaleTotalCount**, **AllScaleTotalNetWgt**, **AllScaleTotalLabelWgt** are incremented or decremented by the serial’s count/label weight/net weight.
- **GoalStatus**:
  - **LimitBy = 'Count'**: If **AllScaleTotalCount >= TargetLevel** → **GoalStatus = 'Complete'**, **SentToServer = no**. If status was **Complete** and **AllScaleTotalCount <= TargetLevel** → **GoalStatus = 'InProcess'**, **SentToServer = no**.
  - **LimitBy = 'NetWgt'**: Same with **AllScaleTotalNetWgt** and **TargetLevel**.
  - **LimitBy = 'LabelWgt'**: Same with **AllScaleTotalLabelWgt** and **TargetLevel**.

**Serial verification** (optional): If **SerialVerification** is enabled, **updatetotals.p** can use **UserBarString** NVP values (e.g. **SLCGoalStatus**, **SLCPrintStatus**, **Verified**) to decide whether to apply the goal update; when verification is off, it always updates. **updatetotals2.p** updates **UserBarString** with **SLCPrintStatus** (Updated/Canceled) and **SLCGoalStatus** (Updated/Canceled) when not using verification.

---

## 11. UpdateGoalStatus (weigh.w)

**Procedure UpdateGoalStatus(ipGoalStatus)**:

- Finds **Goals** by **rowid(Goals) = gGoalsRowid** (exclusive-lock with retry and timeout).
- Sets **Goals.GoalStatus = ipGoalStatus** (e.g. **GoalInProcess** when starting production). Used so the goal is marked In Process when the operator starts working on it.

---

## 12. CheckTargetTime and GoalsProcInTimeSeq

**Procedure CheckTargetTime** (in **w-select-fresh-goals.w**, **w-select-batch-goals.w**, **w-select-goal-btns.w**, **select-order-detail.w**, **order-lines-last-used.w**, **b-open-goals.w**, etc.):

- **Output**: **poOK** (logical). If **gGoalsProcInTimeSeq** is false, **poOK = yes** and no check is done.
- If **gGoalsProcInTimeSeq** is true: find the “earliest” goal by index **TgtDateTimeGoal** where status is Ready, In Process, or Complete. If that goal’s **rowid** is not the one the user selected, show a message that they must select the earliest goal (by target date/time) and set **poOK = no**; otherwise **poOK = yes**.

So when **GoalsProcInTimeSeq** is enabled, the operator must choose goals in target date/time order.

---

## 13. Goals Purge (clearlogs.p)

**File**: `lib/clearlogs.p`

- **Goals-Purge-Days**: Site config option (default **2**). If not present, **SiteConfigOption** and **ConfigOptionMaster** are created with value **2** and description “Delete Goals if older than date, disregard status”.
- **v-goals-clear-days = INT(SiteConfigOption.OptionValue)**.
- For each **Goals** where **(TODAY - Goals.DateStatusChange) > v-goals-clear-days** (exclusive-lock):
  - Delete **CrossRef** where **Application = "Counter-Goal"** and **ID = STRING(Goals.GoalID)**.
  - **DELETE Goals**.
- After that, any **CrossRef** with **Application = "Counter-Goal"** whose **ID** no longer matches a **Goals.GoalID** can be cleaned up (orphan counter removal).

So goals are purged solely by age (**DateStatusChange**), regardless of status, after the configured number of days.

---

## 14. Host: goals-setcount-complete.p

**File**: `host/goals-setcount-complete.p`

- Run by the **Progress Host** as part of its work cycle.
- For each **Goals** where **GoalStatus <> GoalCanceledAtCTS** (no-lock), it:
  - Finds **Production.GoalsDetail** for that GoalID and this scale (gScaleID).
  - Compares SLC **Goals** totals (CurrentCount, CurrentLabelWgt, CurrentNetWgt; AllScale totals) with Production and updates **Goals** and **GoalsHeader**/ **GoalsDetail** so the host and SLC stay in sync. Sets **GoalStatus** to Complete when target is met, or back to In Process when below target.
  - Handles “last case canceled” so a completed goal can be re-opened if totals drop.

So **goals-setcount-complete.p** copies count/weight and status from SLC to the central Production (PDN) and keeps AllScale totals consistent. **lib/updatetotals.p** comments that host/goals-setcount-complete.p copies in one direction (from PDN to CTS) for some flows; the SLC still updates **Goals** locally on add/delete, and the host reconciles.

---

## 15. Goal-Related Session and Config Options

From **gLoginVars.i** / **GlobalAssigns.i**:

- **gGoalsSelectBatch**: If true, use batch date → product → goals flow.
- **gGoalsSelectGoal**: If true, use direct goal selection (e.g. w-select-goal-btns.w).
- **gGoalsSelectOrder**: If true, use order-based selection (w-select-order-btns.w or select-order-browse.w).
- **gGoalsProcInTimeSeq**: If true, **CheckTargetTime** enforces selection in target date/time order.
- **gProduceToWo**: True when the current session is “Produce to Goal” (vs. e.g. Print Label only).

---

## 16. Summary Table

| Concept | Where | Purpose |
|--------|--------|---------|
| **Goals table** | slc.df | One row per production goal: target, product, order, date/time, status, totals, packaging/label overrides. |
| **GoalStatus** | Goals.GoalStatus, prepr.i | Ready (`' '`), InProcess, Complete, Canceled; drives selection and host sync. |
| **LimitBy** | Goals.LimitBy | Count, NetWgt, or LabelWgt for target and totals. |
| **Goal Download** | msg-goal-download.p | Create/update Goals from host JSON; preserve PkgDesc3Size printed counts; RescheduleFlag resets counts. |
| **Goal selection** | s-mainmenu.w, goals/*.w | By batch (date/product), by goal, by order; CheckTargetTime when GoalsProcInTimeSeq. |
| **gGoalID / gGoalsRowid** | gProcessingVars.i, weigh.w | Current goal for weigh session; StartNewProduct sets them. |
| **Serial.GoalID** | createserial.p | Set from Goals when gGoalsRowid is available. |
| **Counter-Goal** | createserial.p, clearlogs.p | CrossRef per GoalID for case counter; deleted when goal is purged. |
| **UpdateTotals / UpdateGoal-Trans** | updatetotals.p | Update Totals and Goals on serial add/delete; status transitions by LimitBy. |
| **updatetotals2.p** | lib | Same plus PkgDesc3Size PrintedCount/PrintedWgt/Printed1/Printed2 and AllScale totals. |
| **GetGoalProdTotal** | weigh.w | Production total, count, target+tolerance, tolerance remaining for display. |
| **UpdateGoalStatus** | weigh.w | Set Goals.GoalStatus (e.g. In Process) when starting production. |
| **Goals-Purge-Days** | clearlogs.p | Delete Goals (and Counter-Goal CrossRef) older than DateStatusChange by N days. |
| **goals-setcount-complete.p** | host | Sync Goals totals and status with Production (PDN). |

---

## 17. Redevelopment Notes

- Replicate **Goals** table (identity, product, order, schedule, status, totals, PkgDesc3Size, packaging/label fields). Keep **LimitBy**, **TargetLevel**, **Tolerance**, **GoalStatus**, **AllScaleTotal***, **Current***, **DateStatusChange**.
- Implement goal download: parse JSON into goal rows; merge into Goals; do not overwrite PrintedCount/PrintedWgt/Printed1/Printed2 (or local progress) from server; handle RescheduleFlag; create BatchOrder when Description matches "Batch:*".
- Implement selection flows: batch (date → product → goals from BatchOrder + Goals), direct goal, order-based; enforce **CheckTargetTime** when **GoalsProcInTimeSeq** is true.
- On “start production” for a goal: set **gGoalID**, **gGoalsRowid** (or equivalent); run **StartNewProduct**; set **GoalStatus** to In Process.
- On serial create: set **Serial.GoalID** (and ItemSerial.GoalID if applicable) from current goal; maintain **Counter-Goal** CrossRef if used.
- On serial add/delete: update **Goals** totals (Current*, AllScaleTotal*), **PkgDesc3Size** (PrintedCount, PrintedWgt, Printed1, Printed2), and **GoalStatus** (Complete when >= TargetLevel, In Process when was Complete and <= TargetLevel); set **SentToServer = no** when status changes.
- **Goals-Purge-Days**: Delete goals where **(TODAY - DateStatusChange) > N**; delete associated **Counter-Goal** (or equivalent) records.
- Replicate host **goals-setcount-complete** logic so the central system and SLC stay in sync on counts and status.
