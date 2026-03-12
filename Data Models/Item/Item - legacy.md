# Item Concept and Usage in the legacy SLC Application

This document describes the **Item** concept in full: what an Item represents, how it relates to Product and Serial, the **ItemModify** and **ItemSerial** tables, and how items are used in the weigh/label flow and in the rest of the application.

---

## 1. Overview

In the SLC, an **Item** is a sub-unit that can be weighed and labeled **within** a case (product). Examples: trays, inner packs, or individual units that make up a case. The **Product** is the case; the **Item** is a component of that case with its own:

- **ItemCode** (1–99999), descriptions, **PackType** (C/F/U/K/V), **MinWgt** / **MaxWgt** / **StdWgt**, **WgtUnits**, and label/date options.
- Optional **ItemModify** row (runtime overrides per item, parallel to **Modify** for product).
- **ItemSerial** records: one per printed item label, linked to the case via **CaseSerialNum** and to the product via **ProductCode**.

The link from production to items is **ProductProcess.Item**: a process step can specify an **Item** (ItemCode). When the operator runs that step, the SLC loads that **Item** (and **ItemModify**) into temp tables **tt-Item** / **tt-ItemModify**, captures item weight, and can create an **ItemSerial** and optionally print an item label. The case **Serial** stores an **ItemWgtList** (e.g. list of item weights for the case). So:

- **Item** = master data for a weighable/labelable sub-unit.
- **ItemModify** = per-item runtime overrides (tare, dates, lot, price, etc.).
- **ItemSerial** = one printed/item weigh event, tied to a case (Serial) and optionally a goal.

---

## 2. Database Schema

### 2.1 Item Table

**Source**: `slc.df` (table **Item**).

| Field | Type | Description / Rules |
|-------|------|----------------------|
| **ItemCode** | integer | Primary key. Range 00001–99999 (VALEXP). Mandatory. |
| **Desc1**, **Desc2** | character (32) | Description lines. Length ≤ 32 (VALEXP). |
| **ShortDesc1**, **ShortDesc2**, **ShortDesc3** | character (8) | Short descriptions. Length ≤ 8 (VALEXP). |
| **PackType** | character (1) | C=Catch, F=Fixed, U=Unipak, K=Key-in, V=Variance. Mandatory. |
| **MinWgt** | decimal | Min weight to allow printing. 0–9999.98. Mandatory. |
| **StdWgt** | decimal | Standard weight for fixed/unipak. Mandatory. |
| **MaxWgt** | decimal | Max weight to allow printing. 0.01–9999.99. Mandatory. |
| **WgtUnits** | character (2) | LB or KG (VALEXP). Mandatory. |
| **WgtRoundOrTruncate** | character (1) | R=round, T=truncate. Mandatory. |
| **WgtRoundDigit** | integer | Round digit. |
| **LabelFile** | character | Label file name. Mandatory. |
| **ItemDateFormat** | character | Item date format. Mandatory. |
| **SellByOffset** | integer | Days from kill date to sell-by. |
| **SellByFormat** | character (1) | Length = 1 (VALEXP). |
| **TareWgt** | decimal | Tare value of packaging. |
| **ExtraTareType** | character (1) | N=None, W=Weight, P=Percent (VALEXP). |
| **WgtTare1**–**WgtTare8**, **PercentTare1**–**PercentTare8** | decimal | Extra tare values. |
| **PostTare** | decimal | Post tare. |
| **VarFormat**, **Offset2**, **ProdDateFormat** | character | Date/format options. |
| **MinLabelWgt**, **MaxLabelWgt** | decimal | Label weight range (e.g. for variance). |
| **ExactItemsPerCaseRequired** | logical | Whether exact item count per case is required. |
| **DateAI**, **DateAIDate** | character | Date AI fields (used in serial/date logic). |

**Index**: **item** (primary, unique) on **ItemCode** ascending.

---

### 2.2 ItemModify Table

**Source**: `slc.df`. **Description**: "Data that can be changed by Item code in the modify screen. The current values are saved by item code."

| Field | Type | Description |
|-------|------|-------------|
| **ItemCode** | integer | Primary key. 00001–99999 (VALEXP). Mandatory. |
| **TareToUse** | integer | Tare number to use when Item.ExtraTareType <> 'N'. |
| **PackDateOffset** | integer | Days to offset pack date. |
| **KillDateOffset** | integer | Days to offset kill date. |
| **VarOffset** | integer | Variable offset from today. |
| **Price** | decimal | Price. |
| **Lot** | character (10) | Lot number. |
| **ModText1**, **ModText2** | character (20) | Length ≤ 20 (VALEXP). |
| **CustName** | character | Customer name. |
| **OrdNum** | character | Order number. |
| **PdnOrdLineNum** | integer | PDN order line. |

**Index**: **ItemModify** (primary, unique) on **ItemCode** ascending.

**Relationship**: One **ItemModify** row per **Item** (same ItemCode). Used at runtime like **Modify** for product: overrides for that item during the session.

---

### 2.3 ItemSerial Table

see [[Item-Serial - legacy]]


---

## 3. Relationships to Other Concepts

### 3.1 Item ↔ Product and ProductProcess

- **ProductProcess** has an **Item** field (integer, format 99999): "Product can have items." When **ProductProcess.Item** is set (non-null), that process step is for that **Item** (ItemCode).
- A **Product** can have **multiple** process steps that reference different **Item**s (or the same item in different steps). So one product can have multiple items; each is identified by **ProductProcess.Item**.
- There is **no** direct Product–Item table; the link is **ProductProcess(ProductCode, ProcessSequence, Item)**. At runtime, for the current product, the code finds all **tt-ProductProcess** rows where **Item <> ?** and for each loads the corresponding **Item** into **tt-Item** (see §5).

### 3.2 Item ↔ ItemModify

- **ItemModify** is keyed by **ItemCode**; at most one row per item. Same pattern as **Modify** for product: runtime overrides (tare, offsets, lot, price, order, etc.) saved by item code and used when creating **ItemSerial** or building item data for the case.

### 3.3 Item ↔ ItemSerial

- **ItemSerial.ItemCode** = **Item.ItemCode**. Each **ItemSerial** row is one weigh/print event for that item; it stores weight, dates, operator, goal, and case link (**CaseSerialNum**, **ProductCode**).

### 3.4 ItemSerial ↔ Serial (Case)

- **ItemSerial.CaseSerialNum** = **Serial.SerialNum** (the case that contains this item).
- **ItemSerial.ProductCode** = **Serial.ProductCode**.
- When the operator finishes a case (or cancels), **Serial** has **ItemWgtList** (e.g. list of item weights). **ItemSerial** rows with **AddOrDel = 'A'** for that **CaseSerialNum** are the item-level records for that case.

### 3.5 ItemSerial ↔ Goals

- **ItemSerial.GoalID** = **Goals.GoalID** when the item was produced in Produce-to-Goal mode (set in **createitemserial.p** from **gGoalsRowid** when available). Same pattern as **Serial.GoalID**.

### 3.6 Item ↔ Goals (Override)

- When **Goals** (Produce to Goal) is present, **weigh-create-tt-item.p** can **UpdateFromGoal**: it overwrites **tt-Item** from Goals (e.g. DateAI, DateAIDate, PkgDesc1/2, ExactItemsPerCaseRequired, UnitTare, wet tare type/weight/percent). So the **first** item for that product in that run can get goal overrides; the Goals architecture does not support multiple distinct items per goal in the DB, so only the first item is updated from goal.

### 3.7 Item and ItemModify ↔ Config / CrossRef

- Item and ItemModify are maintained via **sobjects/v-item.w** and **sobjects/v-item-tare.w**; **ItemModify** can be populated from **Goals** in **weigh-create-tt-itemmodify.p** (CustomerName, OrdNum, PdnOrdLineNum, PackDateOffset, KillDateOffset, Price, VarOffset, etc.) when producing to goal.

---

## 4. Temp Tables and Runtime Flow

### 4.1 TT-Item and TT-ItemModify

**Defined in**: `print/TT-Serial-Source.i`

- **tt-Item**: Like **Item**, with two extra fields **PctTareOverride**, **WgtTareOverride** (for goal/session overrides).
- **tt-ItemModify**: Like **ItemModify**.
- **tt-ItemSerial**: Like **ItemSerial** (used for reprint/source).
- **gTTItemCode**: Global shared int for current item code.

These temp tables hold the **current** item(s) and item modify data for the active product/process. They are filled from **Item** and **ItemModify** (and optionally **Goals**) and used by **CreateItemSerial** and the weigh UI.

### 4.2 Populating tt-Item and tt-ItemModify

- **print/weigh-create-tt-item.p** — **Create-TT-Item**:
  - For each **tt-ProductProcess** where **Item <> ?**, finds **Item** by **ItemCode = tt-ProductProcess.Item**; if not already in **tt-Item**, creates **tt-Item** and buffer-copies **Item**. If **gProduceToWO** and first item, runs **UpdateFromGoal** (Goals → tt-Item overrides).
- **print/weigh-create-tt-itemmodify.p**:
  - Builds **tt-ItemModify** from **ItemModify** (by ItemCode). When producing to goal, can overlay from **Goals** (CustomerName, OrdNum, PdnOrdLineNum, PackDateOffset, KillDateOffset, UnitPrice, VarOffset, etc.).

So the **product process** definition (which steps have **Item** set) determines which items are loaded for the current product.

---

## 5. Item Serial Creation and Weigh Flow

### 5.1 CreateItemSerial (lib/createitemserial.p)

**Purpose**: Create one **ItemSerial** row (add or delete) and optionally format label weight.

- **Inputs**: Add/Del flag, case serial (**piLastSerialCreated**), customer name, weights (extra tare, item post/pre tare, total tare), kill/pack offsets, label weight, lot, net weight, order num/line, item serial number (**piLastItemSerialCreated**), variable offset, **piItemWgtList**.
- **Buffers**: **b-tt-Item**, **b-tt-Product**, **pb-ItemSerial**.
- **Logic**:
  - Uses **Serial-Date** / **Serial-Time** from site config (same as Serial) for consistency.
  - Finds **Goals** by **gGoalsRowid**; sets **ItemSerial.GoalID** if available.
  - **CREATE pb-ItemSerial**; assigns **CaseSerialNum**, **ItemCode** (from b-tt-Item), **ProductCode** (from b-tt-Product), **Operator**, **Shift**, **ScaleNumber**, dates (FalseMidnight adjusted), **NetWgt**, **LabelWgt** (via **FormatBarDec** using Item’s **WgtRoundDigit** and **WgtRoundOrTruncate**), **ItemPostTareWgt**, **ItemPreTareWgt** (from b-tt-Item.TareWgt), **TotalTareWgt**, **ExtraTareWgt**, **Lot**, **Price**, **PdnOrdNum**, **PdnOrdLineNum**, **SellByOffset**, **VarOffset**, **WgtUnits**, **CustomerName**, **Order**, **Sent = no**, **UserBarString = ""**.
  - **VALIDATE pb-ItemSerial**; on error, sets **po-Ok = false** and deletes the record.
  - On success, **gLastItemSerialNum = pb-ItemSerial.SerialNum** (so cancel logic can find the last item serial).

**GetNextItemSerial** (same program): Generates next item serial number (PlantID(2) + ScaleID(2) + JulianDate + "00" + ItemSequence). **ItemSequence** is a session parm (INT); incremented and saved. If an **ItemSerial** with that SerialNum and AddOrDel = "A" already exists, it is deleted (duplicate cleanup).

### 5.2 Role in Weigh Window (print/weigh.w)

- When the current **ProductProcess** step is for an **Item** (tt-ProductProcess.Item <> ?), the weigh window uses **tt-Item** and **tt-ItemModify** for that item. It can run an **item weight** input (capture weight for the item, validate against Item.MinWgt/MaxWgt), then run an output that calls **CreateItemSerial** (and optionally print item label).
- **StartNewProduct** / product init can pass **buffer tt-Item, buffer tt-ItemModify** so the process has the right item context.
- **CancelItemSerial** / **CancelItemSerials**: When the operator cancels one item or the whole case, **CancelItemSerials(ipCaseSerialNum)** finds all **ItemSerial** with **CaseSerialNum = ipCaseSerialNum** and **AddOrDel = 'A'**; for each, if there is not already a matching delete row, it runs **CancelItemSerial** (creates a mirror **ItemSerial** with **AddOrDel = 'D'** and negated weights). Then **gLastItemSerialNum = ''**. So item serials are canceled symmetrically (add + delete rows).

### 5.3 Serial.ItemWgtList

- **Serial.ItemWgtList** (in **lib/createserial.p**) is set from **piItemWgtList** — a list of item weights for the case. It is used for case label tokens: **printlabel.p** supports token **ItemWgtList** and substitutes **STRING(pb-CaseLabelSource.ItemWgtList)**. So the case label can show the list of item weights that belong to that case.

---

## 6. Item and Label / UserBar

- **ItemSerial** has **UserBarString** (NVP); **createitemserial.p** sets it to "" on create. It can be used for item-level barcode/NVP data.
- **Item** has **LabelFile**; item labels use that file when an output step prints an item label.
- **print/userbar.w** can use **tt-ItemLabelSource** (like ItemSerial) for reprint/source.

---

## 7. UI and Maintenance

- **viewers/v-item.w** (or **sobjects/v-item.w**): Item maintenance — browse/edit **Item** (ItemCode, Desc1/2, ShortDesc1/2/3, WgtUnits, MinWgt, MaxWgt, StdWgt, PackType, dates, label file, tares, etc.). Permission **Item.AddChg** (e.g. from **v-item-tare.w** or item viewers) gates add/change.
- **sobjects/v-item-tare.w**: Item tare maintenance (ExtraTareType, WgtTare1–8, PercentTare1–8, TareWgt, PostTare, etc.). Uses **f-get-permission('Item.AddChg')**.
- **viewers/v-item-serial.w**: **ItemSerial** browser (SerialNum, AddOrDel, CaseSerialNum, ProductCode, ItemCode, Operator, Shift, ScaleNumber, weights, dates, GoalID, Lot, etc.). Permission **SystemMenu.EditItemSerial** (or similar) typically gates access.
- **system/import.w**, **system/export.w**: Import/export **Item**, **ItemModify**, **ItemSerial** (temp tables TItem, TItemModify, TItemSerial).
- **system/u-dbclear.w**: **DelItem**, **DelItemModify**, **DelItemSerial** — clear tables (e.g. for testing).

---

## 8. Business Rules Summary

### 8.1 Database (Schema)

- **Item**: ItemCode 1–99999; Desc1/2 length ≤ 32; ShortDesc1/2/3 length ≤ 8; PackType C|F|U|K|V; MinWgt 0–9999.98; MaxWgt 0.01–9999.99; WgtUnits LB|KG; WgtRoundOrTruncate R|T; SellByFormat length 1; ExtraTareType N|W|P; LabelFile, ItemDateFormat, PackType, MinWgt, StdWgt, MaxWgt, WgtUnits, WgtRoundOrTruncate mandatory.
- **ItemModify**: ItemCode 1–99999; ModText1/ModText2 length ≤ 20.
- **ItemSerial**: SerialNum, AddOrDel (A|D), PrintDate, PrintTime mandatory; ItemCode 1–99999; AddOrDel A or D only.

### 8.2 Application

- **ProductProcess.Item** links a step to an Item; only steps with **Item <> ?** load that Item into tt-Item and participate in item weigh/serial creation.
- **ItemSerial** is always tied to a case: **CaseSerialNum** and **ProductCode** must match a **Serial** (case). **GetNextItemSerial** generates unique item serial numbers (Plant+Scale+Julian+"00"+ItemSequence).
- **CancelItemSerials**: For a given case, create delete (D) **ItemSerial** rows that mirror add (A) rows (negated weights) so the case’s item production is canceled.
- **gLastItemSerialNum**: After creating an item serial, set so that cancel/quit can target the last item; after case serial create or cancel-all, clear so the next case does not cancel previous case’s items.
- **Goals**: When producing to goal, the **first** item (first tt-ProductProcess with Item set) gets **UpdateFromGoal** into tt-Item; **tt-ItemModify** can be filled from Goals for that item. **ItemSerial.GoalID** is set when **gGoalsRowid** is available.

---

## 9. Relationship Diagram (Conceptual)

```
Product (case)
    │
    └── ProductProcess (ProcessSequence, Item = ItemCode)
              │
              ├── Item (ItemCode) ──────┬── ItemModify (ItemCode)
              │                         │
              │                         └── ItemSerial (ItemCode, CaseSerialNum, ProductCode, GoalID)
              │                                    │
              └────────────────────────────────────┴──► Serial (SerialNum, ProductCode, ItemWgtList)
```

- **Product** has many **ProductProcess**; some steps have **ProductProcess.Item** set.
- **Item** is referenced by **ProductProcess.Item**; **Item** has one **ItemModify** (by ItemCode).
- **ItemSerial** references **Item** (ItemCode), **Serial** (CaseSerialNum, ProductCode), and optionally **Goals** (GoalID). **Serial.ItemWgtList** aggregates item weights for the case.

---

## 10. Redevelopment Notes

- Implement **Item**, **ItemModify**, **ItemSerial** with same schema and validation (ItemCode range, PackType, weights, WgtUnits, AddOrDel, etc.).
- **ProductProcess** must support an **Item** (ItemCode) field; when present, load that Item (and ItemModify) for the step and allow item weight capture and **ItemSerial** creation.
- **CreateItemSerial** and **GetNextItemSerial** logic: use same serial number format and **ItemSequence** session storage; set **GoalID** from current goal when in produce-to-goal; use **Serial-Date** / **Serial-Time** for consistency with case serial.
- **CancelItemSerials** / **CancelItemSerial**: Create delete rows with negated weights for each add row of the case; clear **gLastItemSerialNum** when starting a new case or after full cancel.
- **Serial.ItemWgtList**: Populate from the list of item weights for the case (e.g. when completing the case); expose **ItemWgtList** token in case label.
- **Goals**: Apply goal overrides to **tt-Item** (and **tt-ItemModify**) for the first item when producing to goal; set **ItemSerial.GoalID** from current goal.
- Item maintenance UI: replicate **v-item.w**, **v-item-tare.w**, and **v-item-serial.w** with same permissions (**Item.AddChg**, **SystemMenu.EditItemSerial**).
