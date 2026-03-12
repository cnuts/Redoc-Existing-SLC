# Modify Concept and Usage in the legacy SLC Application

This document describes the **Modify** concept in full: the Modify table schema, how Modify provides **runtime overrides per product**, how it is loaded into **tt-Modify** and used in serial creation, labels, UserBar, and pallet logic, and how it relates to Product, Serial, Goals, and other concepts.

---

## 1. Overview

**Modify** is a **per-product override table**. It holds values that can be changed at runtime (e.g. on the “modify” screen) without editing the **Product** master. There is at most **one Modify row per ProductCode**. The description in the schema states: “Data that can be changed by product code in the modify screen. The current values are saved by product code. In the future, the key to this table will be Product / Customer / Order.”

Modify is used to:

- **Offset dates**: **PackDateOffset**, **KillDateOffset**, **VarOffset** (days) — passed into **CreateSerial** and used to compute pack date, kill date, and variable date for the serial and barcode.
- **Run-specific data**: **Lot**, **Price**, **CustName**, **OrdNum**, **PdnOrdLineNum** — stored on **Serial** (e.g. **Serial.PdnOrdNum** ← **Modify.OrdNum**, **Serial.Price**, **Serial.Customer**, **Serial.Lot**).
- **Tare selection**: **TareToUse** — which tare number to use when **Product.ExtraTareType** is not 'N' (none).
- **Zulu / NVP**: **ModText1** (and **ModText2**) — name-value pair strings; **ModText1** can hold **ZuluDept** and other NVPs that are merged into **Serial.UserBarString** during serial creation (**ZuluCapture** flow).
- **Pallet**: **CasesPerPallet**, **PalletNum**, **PalletTracking** — used in pallet and weigh UI (e.g. **weigh-enableafteroutput.p**).

When producing to a **goal** (**gProduceToWO**), **weigh-create-modify.p** can **UpdateFromGoal**: goal fields (CasesPerPallet, CustomerName, OrdNum, OrdRefLine, PackDateOffset, UnitPrice, VarOffset) overwrite the corresponding **tt-Modify** fields when the goal supplies values. So Modify is the working set of overrides; it is first loaded from the **Modify** table (by ProductCode), then optionally overridden from **Goals** for the current run.

---

## 2. Modify Table Schema

**Source**: `slc.df`, table **Modify** (LABEL "Modify", DESCRIPTION "Data that can be changed by product code in the modify screen...", DUMP-NAME "modify")

| Field | Type | Description / Rules |
|-------|------|----------------------|
| **ProductCode** | integer | **Primary key.** 00001–99999 (VALEXP). Mandatory. Links to Product. |
| **TareToUse** | integer | Tare number to use when Product.ExtraTareType <> 'N'. Format 99. |
| **PackDateOffset** | integer | Days offset for pack date (e.g. runs over midnight). Format -999. |
| **KillDateOffset** | integer | Days offset for kill date. Format -999. |
| **VarOffset** | integer | Variable offset (days from today) for various dates. Format -999. |
| **Price** | decimal | Price. Format >>,>>9.9994, decimals 2. |
| **Lot** | character | Lot number (alpha-numeric). X(10), max 20. |
| **ModText1** | character | Variable text / NVP string (e.g. ZuluDept). X(20), length ≤ 20 (VALEXP). |
| **ModText2** | character | Variable text. X(20), length ≤ 20 (VALEXP). |
| **CustName** | character | Customer name. x(20), max 40. |
| **OrdNum** | character | Order number. x(20), max 40. |
| **CasesPerPallet** | integer | Cases per pallet. |
| **PalletNum** | character | e.g. PlantID(99) + YDDD + 9999 sequence. x(10), max 20. |
| **PalletTracking** | logical | Whether pallet tracking is on. yes/no. |
| **PdnOrdLineNum** | integer | PDN order line number. Format >>>9. |

**Index**: **Modify** (unique, primary) on **ProductCode** ascending.

---

## 3. tt-Modify Temp Table

**Source**: `print/TT-Serial-Source.i`

- **tt-Modify** is **like Modify** (same fields). Global shared temp table.
- **Find**: **find-tt-modify.i** — `FIND FIRST tt-Modify WHERE tt-Modify.ProductCode = gTTProductCode NO-LOCK NO-ERROR.`
- **Population**: **weigh-create-modify.p** — **Create-Modify** (buffer Goals, buffer Modify, buffer b-tt-Modify). It finds or creates **Modify** for **gTTProductCode**, applies **GlobalKillDateInEffect** (sets KillDateOffset from site config), **ZuluCapture** / **ZuluFromGlobal** (sets ZuluDept in **Modify.ModText1** via NVP), then **buffer-copy modify to b-tt-Modify**. If **gProduceToWO**, it then runs **UpdateFromGoal**, which copies from **Goals** into **b-tt-Modify** (CasesPerPallet, CustomerName, OrdNum, OrdRefLine, PackDateOffset, UnitPrice, VarOffset) when the goal field is not ?. So **tt-Modify** is the active override set for the current product (and optionally goal).

---

## 4. Use in Serial Creation (createserial.p)

**CreateSerial** receives (among other inputs) **piPackDateOffset**, **piKillDateOffset**, **piVariableOffset**. These are passed from the caller (e.g. **case-print.p**) as:

- **if avail b-tt-modify then b-tt-modify.PackDateOffset else 0**
- **if avail b-tt-modify then b-tt-modify.KillDateOffset else 0**
- **if avail b-tt-modify then b-tt-modify.VarOffset else 0**

So pack date, kill date, and variable offset used for the serial (and barcode/dates) come from **Modify** when present.

On the **Serial** record, **createserial.p** assigns:

- **pb-serial.Customer** = **b-tt-modify.CustName** when avail Modify, else ''.
- **pb-serial.Lot** = from hour-code/lot logic, or **b-tt-modify.Lot** when avail Modify, else ''.
- **pb-serial.PdnOrdNum** = **b-tt-modify.OrdNum** when avail, else ''.
- **pb-serial.PdnOrdLineNum** = **b-tt-modify.PdnOrdLineNum** when avail, else 0.
- **pb-serial.Price** = **b-tt-modify.Price** when avail, else 0.
- **pb-serial.VarOffset** = **b-tt-modify.VarOffset** when avail, else 0.

**ZuluCapture**: When **ZuluCapture** is true, **createserial.p** initializes NVP from **b-TT-Modify.ModText1** and iterates NVP pairs; each name-value is written into **pb-serial.UserBarString** via **f-NVP-set-value**. So **Modify.ModText1** is the source of Zulu and other NVPs that end up on the serial’s **UserBarString**.

---

## 5. Use in Case Print and Output Params

**outputparams.i** defines **buffer b-tt-Modify for tt-Modify**. **case-print.p** (and sibling case-print routines) pass **b-tt-Modify** into **CreateSerial** and use **b-tt-modify.PackDateOffset**, **KillDateOffset**, **VarOffset** when calling **CreateSerial**. So every case print that creates a serial uses the current product’s Modify overrides (from tt-Modify).

---

## 6. Use in Weigh Window and Display

- **weigh-displayprodfields.p**: Displays **tt-Modify** values in the weigh UI — Price, Lot, PackDateOffset, KillDateOffset, VarOffset (with computed dates), and **ModText1** (e.g. for NVP display).
- **weigh-enableafteroutput.p**: Uses **b-tt-Modify.PalletTracking** and **b-tt-Modify.CasesPerPallet** for pallet-related UI (e.g. enabling pallet controls, showing cases per pallet).
- **op-hide-btn-setups.p**: Uses **b-tt-Modify.TareToUse** when available for tare button setup.
- **Lot routines** (e.g. **lot-demo.p**, **lot-shapiro.p**): Use **b-tt-Modify.Lot** when available and non-blank as the lot for the run.

---

## 7. Use in UserBar (userbar.p)

**userbar.p** receives **buffer tt-modify** (via find-tt-modify.i). It uses:

- **tt-modify.TareToUse** for UBField **Tare_Number** (single digit).
- **tt-modify.CustName** for **CUSTOMER** (substring by UBFieldCharLoc).
- **tt-modify.VarOffSet** for **VOFFSET** (substring by UBFieldCharLoc).

So UserBar custom barcode/string can include modify-driven values (tare number, customer, variable offset).

---

## 8. Use in Label (printlabel.p)

Label token substitution uses **b-tt-Product** and **pb-CaseLabelSource** (Serial-like). The **Serial** (and thus CaseLabelSource) was created with **Customer**, **Lot**, **PdnOrdNum**, **Price**, **VarOffset** from Modify; so those values appear on the label indirectly via the serial/case source. **Modify** is not passed directly into **PrintLabel**; its effect is through the serial record and **UserBarString** NVPs.

---

## 9. Modify Screen (s-modify.w)

**sobjects/s-modify.w** is the UI for editing **Modify** by product. It:

- Takes **buffer modify** for the current product.
- Displays and edits: **Lot**, **Price**, **CustName**, **OrdNum**, **PackDateOffset**, **KillDateOffset**, **VarOffset**, **TareToUse**, **CasesPerPallet**, **PalletTracking**, **PalletNum**, **ModText1** (and **ModText2**).
- Enforces config/site limits (e.g. **gMaxBackDate** for pack date, messages like **PrintLabels.Modify.Msg.ConfigMaxExceed**).
- Validates **Lot** against **ConfigOptionMaster.AcceptableValues** when lot is restricted (e.g. **PrintLabels.Modify.MissingLots**).
- **Permissions**: Fields can be disabled by permission: **Print.Modify.ZuluCapture**, **Print.Modify.Price**, **Print.Modify.Lot**, **Print.Modify.KillDate**, **Print.Modify.PackDate**, **Print.Modify.VarOffset**, **Print.Modify.Tare** — so the modify screen respects security and only allows editable fields the operator is permitted to change.
- On save, assigns **Modify.*** from the screen values and can update **ModText1** with NVP set-value for ZuluDept. **PalletTracking** is stored (UI may use **PalletTrack** in some widgets; schema field is **PalletTracking**).

Modify is invoked from the Print Label flow; the app ensures a valid **Modify** record exists (creates one if none) before printing and serial creation.

---

## 10. Create-Modify and Goals / Global Kill Date / Zulu

**weigh-create-modify.p**:

- **Create-Modify** finds or creates **Modify** for **gTTProductCode**. If no row exists, it creates one with defaults (e.g. PackDateOffset=0, KillDateOffset=0, Lot='', OrdNum='', Price=0, PalletTracking=no).
- **GlobalKillDateInEffect**: If site config **GlobalKillDateInEffect** is true, **Modify.KillDateOffset** is set to **v-GlobalKillDate - TODAY** so all products use the same kill date.
- **ZuluCapture**: If **ZuluCapture** is true, **ZuluFromGlobal** can set **Modify.ModText1** NVP **ZuluDept** from site config **ZuluGlobalDeptShift** + gShift; otherwise **SetZuluDept** runs and validates **ZuluDept** from **ModText1** against **CrossRef** Application **'ZuluDeptsShift'**, ID = gShift (Descr = comma-separated valid depts). Invalid dept is replaced with the first valid entry and a message is shown.
- After updating **Modify**, **buffer-copy modify to b-tt-modify**.
- **UpdateFromGoal**: If **gProduceToWO**, goal fields (CasesPerPallet, CustomerName, OrdNum, OrdRefLine, PackDateOffset, UnitPrice, VarOffset) are copied into **b-tt-Modify** when the goal supplies non-? values. So **tt-Modify** reflects goal overrides for the current run.

---

## 11. Relationships to Other Concepts

| Concept | Relationship to Modify |
|--------|------------------------|
| **Product** | **Modify.ProductCode** = Product.ProductCode. One Modify row per product. Overrides apply on top of Product (dates, tare, lot, price, customer, order). |
| **Serial** | **Serial** gets **Customer**, **Lot**, **PdnOrdNum**, **PdnOrdLineNum**, **Price**, **VarOffset** from Modify; pack/kill/variable dates use Modify offsets. **Serial.UserBarString** receives NVPs from **Modify.ModText1** (ZuluCapture). |
| **Goals** | When producing to goal (**gProduceToWO**), **UpdateFromGoal** copies goal fields into **tt-Modify** (CasesPerPallet, CustomerName, OrdNum, OrdRefLine, PackDateOffset, UnitPrice, VarOffset). Goal overrides Modify for that run. |
| **Label** | Label content for customer, lot, order, price, dates comes from the serial, which was built using Modify; no direct Modify buffer in PrintLabel. |
| **UserBar** | **userbar.p** uses **tt-Modify** for Tare_Number, CUSTOMER, VOFFSET. |
| **Case print / outputparams** | **b-tt-Modify** passed into **CreateSerial**; PackDateOffset, KillDateOffset, VarOffset passed as parameters. |
| **Weigh UI** | **tt-Modify** drives display (price, lot, date offsets, ModText1) and pallet/tare controls (PalletTracking, CasesPerPallet, TareToUse). |
| **Config / Site** | **GlobalKillDateInEffect**, **GlobalKillDate**, **ZuluCapture**, **ZuluFromGlobal**, **ZuluGlobalDeptShift** + shift, and **gMaxBackDate** affect Modify or its validation. |
| **CrossRef** | **ZuluDeptsShift** + gShift lists valid Zulu dept values for **ModText1** ZuluDept validation. |
| **Permissions** | **Print.Modify.*** permissions control which Modify fields are editable on the modify screen. |

---

## 12. Business Rules Summary

- **ProductCode**: Unique (one Modify per product); 00001–99999; mandatory.
- **ModText1**, **ModText2**: length ≤ 20 (VALEXP). **ModText1** can store NVP data (e.g. ZuluDept); used for **UserBarString** and Zulu validation.
- **PackDateOffset**, **KillDateOffset**, **VarOffset**: Passed into **CreateSerial** and used for pack date, kill date, and variable date; can be overridden by **Goals** when **gProduceToWO** and **UpdateFromGoal** runs.
- **GlobalKillDateInEffect**: When true, **Modify.KillDateOffset** is set from site **GlobalKillDate** so all products share one kill date.
- **ZuluCapture**: When true, **ZuluDept** in **ModText1** is set (from global or shift config) and validated against **CrossRef** ZuluDeptsShift; invalid value is replaced with first valid and user is notified.
- **Goals**: When producing to goal, **tt-Modify** is updated from Goals (CasesPerPallet, CustomerName, OrdNum, OrdRefLine, PackDateOffset, UnitPrice, VarOffset) so the serial and label reflect the goal.
- **Permissions**: Modify screen fields are gated by **Print.Modify.*** (e.g. Price, Lot, KillDate, PackDate, VarOffset, Tare, ZuluCapture).

---

## 13. UI and Maintenance

- **sobjects/s-modify.w**: Modify screen — edit Modify by product; permissions; validation (lot in acceptable values, date limits); save to **Modify** table.
- **weigh-create-modify.p**: Builds **tt-Modify** from **Modify** (find/create), applies GlobalKillDate and Zulu, then **UpdateFromGoal** when producing to goal.
- **weigh-displayprodfields.p**: Fills weigh UI from **tt-Modify** (price, lot, offsets, ModText1).
- **Import/export**: Modify table can be included in system import/export (TModify temp table, key phrase **Modify.ProductCode = TModify.ProductCode**).
- **dbclear**: Can delete Modify (delModify).

---

## 14. Redevelopment Notes

- Implement **Modify** table with **ProductCode** as primary key and all fields (TareToUse, PackDateOffset, KillDateOffset, VarOffset, Price, Lot, ModText1, ModText2, CustName, OrdNum, CasesPerPallet, PalletNum, PalletTracking, PdnOrdLineNum). Enforce length on ModText1/ModText2.
- Implement **tt-Modify** (like Modify); populate from **Modify** by current product; run **UpdateFromGoal** when producing to goal (copy goal fields into tt-Modify).
- **CreateSerial** must accept pack/kill/var offsets and use them for date math; assign **Serial.Customer**, **Serial.Lot**, **Serial.PdnOrdNum**, **Serial.PdnOrdLineNum**, **Serial.Price**, **Serial.VarOffset** from tt-Modify when available. **UserBarString** NVPs from **ModText1** when ZuluCapture (or equivalent) is enabled.
- **GlobalKillDateInEffect** and **ZuluCapture** / **ZuluFromGlobal** / **ZuluDeptsShift** CrossRef logic must be replicated for parity. Modify screen must enforce permissions and validation (lot in acceptable values, date limits).
- **userbar.p** equivalent must read tt-Modify for Tare_Number, CUSTOMER, VOFFSET. Weigh UI and case-print must pass tt-Modify (or its values) where required.
