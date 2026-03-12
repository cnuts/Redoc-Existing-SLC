# Product Concept and Usage in the legacy SLC Application

This document describes the **Product** concept in full: the Product table schema, how Product drives the weigh/label flow via **ProductProcess** and **tt-Product**, how it relates to **Modify**, **Serial**, **Goals**, **Label**, **Item**, **Totals**, and other concepts, and the main business rules.

---

## 1. Overview

A **Product** in the SLC is the **case-level master record** for what is being weighed and labeled. Each product is identified by **ProductCode** (integer 00001–99999) and defines:

- **Descriptions** (Desc1, Desc2, ShortDesc1–3) for labels and UI.
- **Pack type** (C=Catch, U=Unipak, F=Fixed, K=Key-in, V=Variance, H=Hybrid) and **weight rules** (MinWgt, MaxWgt, StdWgt, MinLabelWgt, MaxLabelWgt) used for weight validation and **label weight** calculation (**Set-Label-Wgt**).
- **Label file** name, **weight units** (LB/KG), **round/truncate** and **round digit** for display and barcode.
- **Date/offset** options (SellByOffset, Offset2, VarOffset, ProdDateFormat, SellByFormat, VarFormat, DateAI, DateAIDate) for barcode and label dates.
- **Tare** (BoxTare, ExtraTareType, WgtTare1–8, PercentTare1–8, PostTare).
- **Text1–Text10** for label tokens and UserBar.
- **Pallet** options (ExactItemsPerCaseRequired, ExactCasesPerPalletRequired, CasesPerPallet).

**Product** is the central entity for the production flow: **ProductProcess** defines the steps (inputs/outputs) per product; **Modify** holds runtime overrides per product; **Serial** and **ItemSerial** store **ProductCode**; **Goals** and **Totals** are keyed by product; the **label** template and **UserBar** pull descriptions and weights from Product (or tt-Product). Product data is loaded into the global temp table **tt-Product** (with optional **PctTareOverride**, **WgtTareOverride**) when starting a new product; **tt-Product** is used by **createserial.p**, **printlabel.p**, **userbar.p**, and the weigh window for validation and label data.

---

## 2. Product Table Schema

**Source**: `slc.df`, table **Product** (LABEL "Product", DESCRIPTION "Product Code Definitions", DUMP-NAME "product")

### 2.1 Identity and descriptions

| Field | Type | Rules / Description |
|-------|------|----------------------|
| **ProductCode** | integer | **Primary key.** Format 99999. VALEXP: ProductCode > 0 AND ProductCode <= 99999. Mandatory. "Product Code must be in the range 00001-99999." |
| **Desc1** | character | X(32), length ≤ 32. Description line 1. |
| **Desc2** | character | X(32), length ≤ 32. Description line 2. |
| **ShortDesc1** | character | X(8), length ≤ 8. Short description 1. |
| **ShortDesc2** | character | X(8), length ≤ 8. Short description 2. |
| **ShortDesc3** | character | X(8), length ≤ 8. Short description 3. |

### 2.2 Pack type and weights

| Field | Type | Rules / Description |
|-------|------|----------------------|
| **PackType** | character | X(1). **C**=Catch, **F**=Fixed, **U**=Unipak, **K**=Key-in, **V**=Variance. VALEXP: one of C,F,U,K,V. Mandatory. "Valid pack types: C=Catch, F=Fixed, U=Unipak, K=Key-in, V=Variance." |
| **MinWgt** | decimal | Minimum weight to allow printing. Format >>>9.999. 0–9999.98. Mandatory. |
| **StdWgt** | decimal | Standard weight for fixed/unipak. Mandatory. |
| **MaxWgt** | decimal | Maximum weight to allow printing. 0.01–9999.99. Mandatory. |
| **WgtUnits** | character | X(2). **LB** or **KG**. VALEXP caps = "LB" or "KG". Mandatory. |
| **WgtRoundOrTruncate** | character | **R**=round, **T**=truncate. Mandatory. |
| **WgtRoundDigit** | integer | Round digit for weight display. |
| **MinLabelWgt** | decimal | For PackType V (Variance): if total wgt < MinLabelWgt, label wgt = MinLabelWgt. |
| **MaxLabelWgt** | decimal | For PackType V: if total wgt > MaxLabelWgt, label wgt = MaxLabelWgt. |

(Application logic also supports **PackType H** = Hybrid in **Set-Label-Wgt**; label weight = StdWgt.)

### 2.3 Label and date options

| Field | Type | Description |
|-------|------|-------------|
| **LabelFile** | character | X(8). Label file name (e.g. "Demo"). Mandatory. |
| **ProdDateFormat** | character | Production date format. Mandatory. |
| **SellByOffset** | integer | Days from kill date to sell-by date. |
| **SellByFormat** | character | Sell-by date format. Length = 1 (VALEXP). |
| **VarOffset** | integer | Variable offset. |
| **VarFormat** | character | Variable offset date format. |
| **Offset2** | integer | Second offset for AI dates / barcoding. |
| **DateAI** | character | UCC standard for barcodes – date AI. |
| **DateAIDate** | character | Date used with DateAI. |

### 2.4 Tare

| Field | Type | Description |
|-------|------|-------------|
| **BoxTare** | decimal | Tare value of packaging. |
| **ExtraTareType** | character | **N**=None, **W**=Weight, **P**=Percent. VALEXP. |
| **WgtTare1** … **WgtTare8** | decimal | Weight tare values. |
| **PercentTare1** … **PercentTare8** | decimal | Percentage tare values. |
| **PostTare** | decimal | Usually ice or packing material. |

### 2.5 Text and pallet

| Field | Type | Description |
|-------|------|-------------|
| **Text1** … **Text10** | character | X(64), length ≤ 64. Free-form text for labels and UserBar. |
| **ExactItemsPerCaseRequired** | logical | Whether exact item count per case is required. |
| **ExactCasesPerPalletRequired** | logical | Whether exact cases per pallet is required. |
| **CasesPerPallet** | integer | Cases per pallet. |

**Index**: **prod** (unique, primary) on **ProductCode** ascending.

---

## 3. tt-Product Temp Table

**Source**: `print/TT-Serial-Source.i`

- **tt-Product** is **like Product** with optional extra fields, e.g. **PctTareOverride**, **WgtTareOverride** (and any other override fields used in the app).
- **tt-Product** is the in-memory copy of the **current** product used during a weigh/print session. It is populated when the operator starts or selects a product (**Create-TT-Product** in **weigh-create-tt-product.p**).
- **gTTProductCode** holds the current product code; **ProductProcess**, **Modify**, and **Item** temp tables are keyed by this product.

**Population**: **weigh-create-tt-product.p** — **Create-tt-product** (buffer Product, buffer b-tt-Product, buffer Goals). If **Product** is available, **buffer-copy Product to b-tt-Product** and set overrides (PctTareOverride, WgtTareOverride). For **family processing**, if a ProductProcess with "family" in the routine exists for the product (or for **ModeProdCodeDefault**), **tt-Product** MinWgt/MaxWgt can be overridden from **CrossRef** ("Family-" + ProductCode) and linked **Product** records. So **tt-Product** may differ from the database **Product** when family or overrides apply.

---

## 4. ProductProcess Link

**ProductProcess** (slc.df) has **ProductCode** (integer 00001–99999, same VALEXP as Product). Each row is one **process step** for that product (e.g. Case-Weight, CaseOutput, GetProduct, ItemOutput). **weigh-create-tt-prodprocess.p** builds **tt-ProductProcess** from **ProductProcess** for **gTTProductCode**; **OutputLabel** is set from **b-TT-Product.LabelFile**. So:

- **Product** defines **LabelFile**; the **output** step’s **OutputLabel** comes from **Product.LabelFile** (unless overridden per step).
- **Product** does not store process sequence; **ProductProcess** holds **ProcessSequence**, **ProcessID**, **Routine**, **Device**, **Item**, etc., per product.

Validation (**validateprodprocess.p**) ensures every **ProductProcess** step references a valid **Device** (and **AlternateDevice** when set). So Product + ProductProcess define *what* is produced and *how* (steps and devices).

---

## 5. Modify (Runtime Overrides)

**Modify** has **ProductCode** (FK to Product). For each product, zero or one **Modify** row can supply runtime overrides: **TareToUse**, **PackDateOffset**, **KillDateOffset**, **VarOffset**, **Price**, **Lot**, **CustName**, **OrdNum**, **PdnOrdLineNum**, **ModText1**, **ModText2**, etc. When creating a serial, **createserial.p** uses **b-tt-Modify** (and **b-tt-Product**) for dates, lot, price, order, and Zulu/NVP data. So **Product** is the master; **Modify** overrides selected fields at runtime without changing the Product record.

---

## 6. Serial, ItemSerial, Totals, Goals

- **Serial.ProductCode** and **ItemSerial.ProductCode** reference the product for that case or item serial. **Serial** is created with **ProductCode** from the current product (tt-Product); weight and label rules (MinWgt, MaxWgt, PackType, etc.) come from **tt-Product** and **tt-Modify**.
- **Totals** is keyed by **Shift**, **Scale**, **PdnDate**, **ProductCode**; aggregates (LocalCount, LocalLabelWgt, etc.) are updated when serials are added/deleted for that product.
- **Goals** has **ProdCode** (integer); goal selection and production-to-goal are by product. **Goals** can override **LabelFile** (e.g. in **v-tt-product.w**); when producing to a goal, **Serial.GoalID** and **ItemSerial.GoalID** are set from the current goal.

So **Product** (and **ProductCode**) ties together process definition (ProductProcess), overrides (Modify), production records (Serial, ItemSerial), aggregates (Totals), and targets (Goals).

---

## 7. Label and Set-Label-Wgt

- **Product.LabelFile** (and **Goals.LabelFile** when present) determines the label template; **ProductProcess.OutputLabel** is set from **Product.LabelFile** when building tt-ProductProcess.
- **Set-Label-Wgt** (**print/set-label-wgt.p**) takes **ip-PackType**, **ip-TotalProductWgt**, **ip-StdWgt**, **ip-MinLabelWgt**, **ip-MaxLabelWgt** and returns **op-LabelWgt**:
  - **C** (Catch): label wgt = total product wgt.
  - **U** (Unipak): label wgt = StdWgt.
  - **F** (Fixed): label wgt = min(TotalProductWgt, StdWgt).
  - **V** (Variance): clamp total to [MinLabelWgt, MaxLabelWgt].
  - **H** (Hybrid): label wgt = StdWgt.
- **printlabel.p** uses **b-tt-Product** for descriptions (DESCRIPTION1OF2, SHORT1OF3, …), **Text1–Text10**, **WgtRoundDigit**, **WgtRoundOrTruncate**, **MinLabelWgt**, **MaxLabelWgt**, **PackType**, and product-level barcode/date logic. So **Product** (via tt-Product) drives both *which* label file is used and *how* weights and text are formatted on the label.

---

## 8. Item and ProductProcess.Item

- **Item** is a sub-unit of a case; **ProductProcess** can have an **Item** (ItemCode) for steps that weigh or print item labels. The **product** remains the case product; **ItemSerial** stores both **ProductCode** and **ItemCode**. So **Product** is the case product; **Item** is optional component master data, linked through **ProductProcess.Item** for item steps.

---

## 9. CrossRef and Family Processing

- **CrossRef** is used with **Product** in several ways: **Product-MFGID** (CrossRef.ID = product code, Descr = MFGID override); **Family-** + ProductCode for family processing (CrossRef entries point to other product codes for MinWgt/MaxWgt overrides in **weigh-create-tt-product.p**).
- **Family processing**: When a ProductProcess routine contains "family", the app can override **tt-Product.MinWgt** and **tt-Product.MaxWgt** from **CrossRef** "Family-&lt;ProductCode&gt;" and the corresponding **Product** rows, so one “family” product code can use min/max from other products in the group.

---

## 10. Weigh Flow Summary

1. Operator selects or scans a product (or selects a goal/product). **Product** (and optionally **Goals**) are loaded.
2. **Create-TT-Product** (weigh-create-tt-product.p) fills **tt-Product** from **Product**, applies family overrides from **CrossRef** if applicable.
3. **Create-TT-ProductProcess** (weigh-create-tt-prodprocess.p) builds **tt-ProductProcess** for **gTTProductCode**; **OutputLabel** is set from **tt-Product.LabelFile**.
4. **Create-TT-Modify** (and **Create-TT-Item** / **Create-TT-ItemModify** when item steps exist) fill modify/item temp tables for the current product.
5. Weigh window runs steps from **tt-ProductProcess**. Weight is validated against **tt-Product.MinWgt** / **MaxWgt**; **Set-Label-Wgt** uses **tt-Product.PackType** and weight limits to compute label weight.
6. **createserial.p** creates **Serial** with **ProductCode** from **b-tt-Product**, using **b-tt-Product** and **b-tt-Modify** for dates, lot, tare, barcode, and **UserBarString** NVPs.
7. **case-print.p** (or sibling) uses **tt-Product**, **tt-CaseLabelSource**, **Goals** and calls **PrintLabel** with **ProductProcess.OutputLabel** (from Product.LabelFile) and **ProductProcess.Device**.

So **Product** (and **tt-Product**) are central to product selection, weight rules, label file, serial creation, and label content.

---

## 11. Relationships to Other Concepts

| Concept | Relationship to Product |
|--------|--------------------------|
| **ProductProcess** | Steps are per **ProductCode**; **OutputLabel** from **Product.LabelFile**; **Device**, **Item**, **Routine** per step. |
| **Modify** | **Modify.ProductCode** = Product; runtime overrides (tare, dates, lot, price, order) applied with **Product** in serial/label creation. |
| **Serial** | **Serial.ProductCode** = current product; weight and label data from **tt-Product** and **tt-Modify**. |
| **ItemSerial** | **ItemSerial.ProductCode** = case product; item steps may also reference **Item** (ItemCode). |
| **Goals** | **Goals.ProdCode** = product; goal selection and production-to-goal by product; **Goals.LabelFile** can override product label file. |
| **Totals** | **Totals.ProductCode**; aggregates updated when serials for that product are added/deleted. |
| **Label** | **Product.LabelFile**, **MinLabelWgt**, **MaxLabelWgt**, **PackType**, descriptions, **Text1–10**, **WgtRoundDigit**, **WgtRoundOrTruncate** feed token substitution and **Set-Label-Wgt**. |
| **Item** | **ProductProcess.Item** links product step to **ItemCode**; case remains **Product**; **Item** has its own PackType/weights/label file. |
| **UserBar** | **userbar.p** uses **tt-Product** for PACK_TYPE, descriptions, weights, Text1–10, WgtUnits, etc. |
| **BatchOrder** | Can reference product (e.g. ProdCode) for batch/goal context. |
| **Locals** | **Locals.ProductCode** for display/local code mapping. |
| **CrossRef** | **Product-MFGID** (product-specific MFGID); **Family-** + ProductCode for family min/max overrides. |
| **Device** | **ProductProcess.Device** per step; Product does not store device directly. |

---

## 12. Business Rules Summary

- **ProductCode**: 00001–99999; mandatory; unique (primary key). Same range enforced on **Serial**, **ItemSerial**, **ProductProcess**, **Modify**, **Totals**, **Goals** (ProdCode), **Locals**, **PalletDtl** where they store product.
- **PackType**: C, F, U, K, or V (schema); application also supports **H** (Hybrid) in **Set-Label-Wgt**. Mandatory.
- **MinWgt**, **MaxWgt**, **StdWgt**: Mandatory; used for weight validation and label weight calculation. **MinLabelWgt** / **MaxLabelWgt** used for PackType V (and optionally elsewhere).
- **LabelFile**: Mandatory; drives **ProductProcess.OutputLabel** and label template resolution.
- **WgtUnits**, **WgtRoundOrTruncate**: Mandatory. **Text1–10**: length ≤ 64. **Desc1/2**: length ≤ 32. **ShortDesc1–3**: length ≤ 8.
- **Product** must exist before a weigh/print session can use it; **tt-Product** is built from **Product** (and family overrides). **ProductProcess** must exist for the product (or defaults like 99999) so that steps are available.

---

## 13. UI and Maintenance

- **viewers/v-products.w**: Product maintenance (SmartViewer/browse over **Product**); **ProductCode** validation, **tt-Product** buffer copy for related operations; **Locals** and product code checks.
- **viewers/v-products-text.w**: Product text fields (Text1–10, etc.) maintenance.
- **system/v-tt-product.w**: Temp/product maintenance in system context; can set **fiLabelFile** from **Goals.LabelFile** when editing under a goal.
- **Import/export**: Product table can be included in system import/export.
- Product is referenced in goal selection, batch selection, and weigh “start new product” / “get product” flows.

---

## 14. Redevelopment Notes

- Implement **Product** table with **ProductCode** (1–99999) as primary key and all fields (descriptions, PackType, weights, label file, date options, tare, text, pallet flags). Enforce VALEXP/VALMSG equivalents.
- Implement **tt-Product** (and **gTTProductCode**) as the current-product snapshot; populate from **Product** when starting/selecting product; support **PctTareOverride** / **WgtTareOverride** and family overrides from **CrossRef** if needed.
- **ProductProcess** must be keyed by **ProductCode**; **OutputLabel** from **Product.LabelFile**; **Set-Label-Wgt** and weight validation must use **Product** (tt-Product) PackType and weight fields.
- **Modify** keyed by **ProductCode**; apply overrides in serial creation and label flow. **Serial** and **ItemSerial** store **ProductCode**; **Totals** and **Goals** (ProdCode) aggregate or target by product.
- **Label** and **UserBar** must receive product (or tt-Product) for descriptions, Text1–10, weights, and label file. **Family** processing and **Product-MFGID** **CrossRef** logic are optional but documented for parity.
