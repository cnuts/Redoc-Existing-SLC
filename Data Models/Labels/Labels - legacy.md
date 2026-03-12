# Label Concept and Usage in the legacy SLC Application

This document describes the **Label** concept in full: how labels are defined, how they are filled with data (token substitution), how they are sent to printers, and how they relate to Serial, Product, Goals, Modify, Item, Device, and other concepts.

---

## 1. Overview

A **label** in the SLC is a **printable output** (case label or item label) produced by:

1. **Template file**: A text/binary file (e.g. `labels/*.lbl`) containing **bracketed tokens** such as `[SERIAL]`, `[BARCODE]`, `[NETWEIGHT]`, `[PRODUCT]`, `[BatchID]`, `[ProcessOrder]`.
2. **Data source**: At print time, a **case label source** (e.g. **tt-CaseLabelSource**, which is like **Serial** plus **CaseSeq**, **MarkDownPrice**, **PkgDesc3**) and optionally **Goals**, **tt-Product**, **tt-ItemLabelSource** supply the values.
3. **Token substitution**: **lib/printlabel.p** reads the template, replaces each `[TokenName]` with the corresponding value (from Serial/case source, Product, Goals, UserBarString NVP, CrossRef, or site config), and builds the final output.
4. **Output**: The resulting stream (raw/binary) is sent to a **printer** via **gh_SendOutput** (**WriteToDevice**) using a **COM handle** and **Device ID** (from **ProductProcess.Device** or **Device** table).

Labels are used for **case labels** (one per Serial/case), **cancel labels** (when a serial is canceled), **item labels** (when printing item-level labels), and **reprint**. The **label file** name comes from **Product.LabelFile**, **Item.LabelFile**, **Goals.LabelFile**, or **ProductProcess.OutputLabel** (and related logic); the file is resolved under a **labels/** directory or via **SEARCH**.

---

## 2. Label File and Path

### 2.1 Where the Label File Name Comes From

- **Product.LabelFile**: Default label file for the product (e.g. "Demo", "Prod1"). Stored in Product table; 8-char format typical.
- **Item.LabelFile**: Label file for item labels (Item table).
- **Goals.LabelFile**: When producing to goal, the goal can override the label file (e.g. in **v-tt-product.w**, `fiLabelFile = Goals.LabelFile` when available).
- **ProductProcess.OutputLabel**: Set from **Product.LabelFile** when building the product process temp table (**weigh-create-tt-prodprocess.p**). The **output** step that prints the case label uses **b-tt-ProductProcess.OutputLabel** (or **Device**-specific file) to determine which template to use.
- **Reprint**: **tt-Reprint-Controls.LabelFile** or user-selected file in reprint UI (e.g. **reprint-case-lbl.w**).

### 2.2 Path Resolution

- **print/set-labels.p**: Builds paths as **"labels/" + TRIM(op-LabelFile) + ".lbl"** (and **.1st**, **.2nd**, **.can**). Uses **FILE-INFO:FILE-NAME** to resolve the main template; if the file exists, **op-LabelFile** (and **op-FirstLabel**) is set to that full path. **op-QuickLabel** is the **.2nd** file if present, else **op-FirstLabel**. **op-CancelLabel** is **"labels/cancel.lbl"** or the **.can** file if present. Callers (e.g. **case-print.p**) pass **b-tt-ProductProcess.OutputLabel** as the base name; **Set-Labels** returns the path(s) used for **PrintLabel**.
- **lib/printlabel.p** — **PrintLblInits** (internal procedure):
  - **Input**: **ip-AddOrDelete**, **ip-LabelFile** (file name or path, e.g. `labels/demo.lbl` or `demo`).
  - **Output**: **poPathNameToOpen** = **SEARCH(ip-LabelFile)**. If not found, logs and shows message "Label File Not Found" / "Missing Label OR folder was moved"; returns and **PrintLabel** exits without printing.
  - So the **caller** typically passes a path like **"labels/demo.lbl"** or a name that **SEARCH** can resolve (e.g. with **labels/** prefix added elsewhere). If **piLabelFile** is null (e.g. product has no label file), **PrintLblInits** can return ? and **chrPathNameToOpen = ?** causes **PrintLabel** to return.

### 2.3 File Format and Reading

- The template is read **BINARY UNBUFFERED NO-CONVERT** (**input stream s-Label from value(chrPathNameToOpen)**). So labels can be text or binary (e.g. Zebra ZPL); **PrintLabel** processes character by character (READKEY), detects `[` and `]`, and substitutes the token between them. After substitution, the result is built in **vRawLabel** (RAW) and sent to the device.
- **Bartender**-style labels: Comments in printlabel.p note that labels created through Bartender may have images encoded and field names with a table name prefix in brackets; the code strips a leading "TableName." from the token (e.g. **ENTRY(2, chrStringToMatch, ".")**) when **NUM-ENTRIES(chrStringToMatch, ".") >= 2**.

---

## 3. Case Label Source (tt-CaseLabelSource)

**Defined in**: `print/TT-Serial-Source.i`

- **tt-CaseLabelSource** is **like Serial** with three extra fields: **CaseSeq** (int), **MarkDownPrice** (dec), **PkgDesc3** (c).
- **Index**: **CaseLabelSource-Index** primary unique on **CaseSeq**.
- **Purpose**: Single record that holds all Serial-like data (and Goals-derived overrides) for **one** case label print. Filled by the print routine (e.g. **case-print.p**) from **Serial** (buffer-copy) and optionally **Goals** (MarkDownPrice, PkgDesc3). **PrintLabel** receives **buffer pb-CaseLabelSource** for **tt-CaseLabelSource**; most tokens pull from **pb-CaseLabelSource** (e.g. SerialNum, NetWgt, LabelWgt, PackDate, KillDate, Lot, Price, PdnOrdNum, Shift, Scale, Operator, Customer, GoalID, UserBarString, ItemWgtList, tare fields, PrintDate, PrintTime).

So the **label** is bound to **one case (Serial)** through **tt-CaseLabelSource**; that buffer is the primary **case label source** for token substitution.

---

## 4. Token Substitution — Data Sources and Examples

**Procedure**: **PrintLabel** in **lib/printlabel.p**

- **Parameters**: **piAddorDelete** (A/D), **piLabelFile**, **pih_Printer** (COM-HANDLE), **piDeviceID**; **buffer b-tt-Product**, **buffer pb-ItemLabelSource**, **buffer pb-CaseLabelSource**, **buffer pb-Goals**.
- **Flow**: **PrintLblInits** → get path; **FormatBarDec** for barcode weight; barcode/human-readable from **UserBarString** NVP "Barcode"; open stream; **100-reformat-lbl** loop: read character by character, when `[`…`]` found, **chrStringToMatch** = token name (with optional "TableName." stripped); **CASE (chrStringToMatch)** maps to replacement text; replacement is appended to **vRawLabel**. Optional override format via trailing hyphen (e.g. **Token-0**). At end, **WriteToDevice** (gh_SendOutput) sends **vRawLabel** and **piDeviceID** to the printer.

### 4.1 Tokens and Their Sources (Representative List)

| Token | Source | Notes |
|-------|--------|--------|
| **PRODUCT** | pLblProduct(ProductCode) | Product code or description per product logic. |
| **SERIAL** | pb-CaseLabelSource.SerialNum | Case serial number. |
| **LAST4** | pb-CaseLabelSource.SerialNum | Last 4 digits of serial. |
| **UNIQUE** | Session parm UNIQUENUM | Unique number. |
| **LOT** | pb-CaseLabelSource.Lot | Lot. |
| **ORDER** | pb-CaseLabelSource.PdnOrdNum | Order number. |
| **PLANT** | gPlantID | Plant ID. |
| **PRICE** | pb-CaseLabelSource.Price | Price. |
| **SCALE** | pb-CaseLabelSource.Scale | Scale number. |
| **BARCODE** | UserBarString NVP "Barcode" (or CreateBarCode) | Barcode string. |
| **HUMANREADABLE** | Same as Barcode human-readable. | |
| **DESCRIPTION1OF2**, **DESCRIPTION2OF2**, **DESCRIPTION1OF4**…**4OF4** | b-tt-product.Desc1, Desc2 | Product descriptions. |
| **SHORT1OF3**…**SHORT3OF3** | b-tt-product.ShortDesc1–3 | Short descriptions. |
| **PRINTTIME**, **PRINTDATE** | pb-CaseLabelSource.PrintTime, PrintDate | Print date/time. |
| **PRODDATE**, **PACKDATE** | pb-CaseLabelSource (pLblProdDates) | Pack/production dates with format. |
| **KILLDATE**, **SellByDate** | pb-CaseLabelSource, Goals, Product (pLblProdDates / printlabel-when.i) | Kill/sell-by with overrides. |
| **NETWEIGHT**, **GROSSWEIGHT** | pb-CaseLabelSource.NetWgt, TotalTareWgt | Weights. |
| **LABELWEIGHT** | pb-CaseLabelSource.LabelWgt (or product min/max/format) | Label weight. |
| **XLABELWEIGHT** | Unit conversion (KG↔LB) per WgtUnits. | |
| **TAREWEIGHT**, **BOXTARE** | pb-CaseLabelSource.TotalTareWgt, BoxTare | Tare weights. |
| **CasePostTareWgt**, **ContainerPreTareWgt**, **ItemPostTareWgt**, **ItemPreTareWgt** | pb-CaseLabelSource | Tare components. |
| **ItemWgtList** | pb-CaseLabelSource.ItemWgtList | Item weights list for case. |
| **MOISTURETARE** | pb-CaseLabelSource.ExtraTare | Extra tare. |
| **TEXT01**…**TEXT10** | b-tt-product.Text1–Text10 | Product text fields. |
| **TIMELOT** | PrintTime + Lot or variable offset text (site config). | |
| **SHIFT** | pb-CaseLabelSource.Shift | Shift. |
| **Goalid** | pb-CaseLabelSource.GoalID | Goal ID. |
| **CUSTOMER** | pb-CaseLabelSource.Customer | Customer. |
| **USER** | pb-CaseLabelSource.Operator | Operator. |
| **VERSION** | gVersion | App version. |
| **PKGDESC3** | pb-CaseLabelSource.PkgDesc3 | Package description 3. |
| **BatchID** | UserBarString NVP "BatchID" | 4-digit batch ID from batch-id.w. |
| **ProcessOrder**, **PackOrder** | UserBarString NVP "ProcessOrder", "PackOrder" | MII/batch process/pack order. |
| **Type2UPC_A_BarCode**, **BARCODE_UPC_A_CUSTOM** | build-type2-upc-a-barcode.p, build-upc-a-barcode.p | Custom barcode builds (CaseLabelSource, ItemLabelSource). |
| **SkidWeight**, **PartialFillWeight**, **TopOffNet**, **CalculatedGrossWeight**, **SerialNetWeight**, **ProductBoxTare**, **ProcessingMode** | UserBarString NVP or calculations | Processing-mode and weight variants. |
| **USERBARA**…**USERBARZ** | lib/userbar.p | User-defined barcode (UBHeader/UBDetail from Serial, Product, Goals, etc.). |
| **FALSEMIDNIGHT** | gFalseMidNight | False midnight time. |
| **COUNTER** | Counter-CrossRef or similar | Counter value (e.g. RM 6555). |
| **ProductText11**…**ProductText99** (and subfields **ProductText21A**…**Z**) | CrossRef Product + NVP (GetProductText) | Product text 11–99 from CrossRef; optional subfields. |
| **units**, **xunits** | PlblUnits | Units display. |

Tokens not found in the **CASE** are left as-is or yield blank (depending on logic). **Override format**: token name can have suffix **-n** (e.g. **-0**) for format override; **chrOverrideFormat** is used in formatting.

---

## 5. Label Weight (Set-Label-Wgt)

**print/set-label-wgt.p** — **Set-Label-Wgt**

- **Inputs**: **ip-PackType** (C/F/U/K/V/H), **ip-TotalProductWgt**, **ip-StdWgt**, **ip-MinLabelWgt**, **ip-MaxLabelWgt**.
- **Output**: **op-LabelWgt**.
- **Rules**:
  - **C** (Catch): **op-LabelWgt = ip-TotalProductWgt**.
  - **U** (Unipak): **op-LabelWgt = ip-StdWgt**.
  - **F** (Fixed): **op-LabelWgt = MIN(ip-TotalProductWgt, ip-StdWgt)**.
  - **V** (Variance): If total ≤ MinLabelWgt → MinLabelWgt; if total ≥ MaxLabelWgt → MaxLabelWgt; else total.
  - **H** (Hybrid): **op-LabelWgt = ip-StdWgt**.
  - **Otherwise**: **op-LabelWgt = ip-TotalProductWgt**.

This value is the **label weight** printed on the label and stored on **Serial.LabelWgt** (and used in **FormatBarDec** for barcode). So the **label** concept ties directly to **Product.PackType** and product min/max/std weight.

---

## 6. Set-Labels (First, Quick, Cancel)

**print/set-labels.p** — **Set-Labels**

- **Input**: **ip-LabelFile** (base name, e.g. "Demo" or "DEFAULT").
- **Outputs**: **op-FirstLabel**, **op-QuickLabel**, **op-CancelLabel**, **op-LabelFile**.
- **Logic**: Resolves **labels/** + Trim(LabelFile) + **.lbl** (main), **.1st** (first), **.2nd** (quick), **.can** (cancel). If **.1st** exists, **op-FirstLabel** = that path; if **.2nd** exists, **op-QuickLabel** = that path else **op-FirstLabel**; **op-CancelLabel** = **labels/cancel.lbl** or **.can** if present.

Used by case-print family and case-batch-serial to choose the correct template for first label, quick (subsequent) label, and cancel label.

---

## 7. Device and Printer Output

- **Device** table (slc.df): **DeviceID** (primary), plus communication/settings (e.g. ComPort, BaudRate). **ProductProcess.Device** holds the **DeviceID** (character) for the step that prints (e.g. case label).
- **PrintLabel** receives **pih_Printer** (COM-HANDLE) and **piDeviceID**. The COM handle is the printer object (e.g. from **sobjects/s-setupprinter.w** or the weigh/output flow). **WriteToDevice** in **gh_SendOutput** sends the raw label (**vRawLabel**) to that handle/device.
- **gPrinterAttached**: When false, **PrintLabel** does not call **WriteToDevice** (no physical print). So the **label** is still built and can be logged or tested.

---

## 8. Invocation Flow (Case Label)

1. **Product process** runs an **output** step (e.g. **case-print.p**, **case-label.p**, **case-batch-serial.p**). The step has **Device** and **OutputLabel** (label file).
2. **Serial** is created (or already exists for reprint/cancel). **tt-CaseLabelSource** is filled from **Serial** and optionally **Goals** (MarkDownPrice, PkgDesc3).
3. **Set-Label-Wgt** is run to get **vLabelWgt** from PackType and weights; **Serial.LabelWgt** is set in **CreateSerial** from that value.
4. **Set-Labels** (or equivalent) may be used to get **vLabelFile** (e.g. **labels/demo.lbl**).
5. **RUN PrintLabel in gh_PrintLabel** with **'A'** or **'D'**, **vLabelFile**, **ipCtlOutputDevice**, **Device**, **buffer tt-Product**, **buffer tt-ItemLabelSource**, **buffer tt-CaseLabelSource**, **buffer Goals**.
6. **PrintLabel** calls **PrintLblInits** → **SEARCH(piLabelFile)**; builds barcode from UserBarString; reads template; substitutes tokens; **WriteToDevice(vRawLabel, piDeviceID)**. Repetitions (e.g. **b-tt-ProductProcess.Repetitions**) cause multiple **PrintLabel** calls for the same case.

**Cancel label**: Same flow but **piLabelFile** = **"labels/cancel.lbl"** (or .can), **piAddorDelete** = **'D'**, and **tt-CaseLabelSource** holds the serial being canceled.

---

## 9. Relationships to Other Concepts

| Concept | Relationship to Label |
|--------|------------------------|
| **Serial** | Primary data for case label: SerialNum, NetWgt, LabelWgt, dates, Lot, Price, Operator, Shift, Scale, GoalID, UserBarString, ItemWgtList, tares. **tt-CaseLabelSource** is Serial-like. |
| **Product** | **Product.LabelFile** = default template name. **Product.PackType**, **MinLabelWgt**, **MaxLabelWgt**, **StdWgt** drive **Set-Label-Wgt**. **Product** descriptions and Text1–10 feed tokens. **Product** CrossRef (e.g. Product-MFGID, Product text 11–99) feeds tokens. |
| **Goals** | Can override **LabelFile**; **Goals.MarkDownPrice**, **Goals.PkgDesc3** (and other fields) can populate **tt-CaseLabelSource** or token logic (e.g. SellByDate from Goals when producing to goal). |
| **Modify** | Runtime overrides (dates, lot, price, etc.) are applied to Serial/case data before or during **CreateSerial**; that data is then in **tt-CaseLabelSource** for the label. |
| **Item / ItemSerial** | **ItemWgtList** on Serial (and CaseLabelSource) is the list of item weights for the case. **ItemLabelSource** buffer is passed to **PrintLabel** for tokens that use item data (e.g. Type2UPC, BARCODE_UPC_A_CUSTOM). |
| **Batch** | **BatchID** token from **UserBarString** NVP (set by batch-id.w). **ProcessOrder**, **PackOrder** from UserBarString (MII/batch flow). |
| **Device** | **ProductProcess.Device** = DeviceID for the printer; **piDeviceID** and COM handle sent to **WriteToDevice**. |
| **CrossRef** | **Product** CrossRef for ProductText11–99, TextWeight, MFGID; **Counter-Goal** for COUNTER. **userbar.p** uses UBHeader/UBDetail for USERBARA–Z. |
| **Site config** | **gVersion**, **gFalseMidNight**, **gPlantID**, variable offset text, etc. feed tokens. |
| **UserBarString (NVP)** | Stored on Serial; holds Barcode, BatchID, ProcessOrder, PackOrder, ProcessingMode, SkidWeight, PartialFillWeight, etc. Populated during serial creation and by product process/MII. |

---

## 10. Sample and Cancel Labels

- **Sample label**: When **gv-SampleLabel** is true, **PrintLabel** can insert "SAMPLE" on the barcode line (Zebra or other format) and use serial prefix "00" so the serial is clearly a sample. Sound and logic in **PrintLabel** and **case-print.p** support sample mode.
- **Cancel label**: Printed when a serial is canceled (AddOrDel = 'D'). **weigh.w** (and similar) runs **PrintLabel** with **'D'** and **"labels/cancel.lbl"** (or .can), and **tt-CaseLabelSource** filled from the serial being canceled. No new Serial row for the cancel print; the cancel label is just a physical record that a case was voided.

---

## 11. Item Labels

- **Item** has **LabelFile**; when an **item** label is printed, the template can be the item’s label file. **pb-ItemLabelSource** (tt-ItemLabelSource, like ItemSerial) is passed to **PrintLabel** for tokens that need item-level data (e.g. **Type2UPC_A_BarCode**, **BARCODE_UPC_A_CUSTOM**). Case label and item label share the same **PrintLabel** procedure; the same tokens are available, but **ItemLabelSource** is used when the token logic requires it.

---

## 12. Userbar (USERBARA–USERBARZ)

**lib/userbar.p**: For tokens **USERBARA** through **USERBARZ**, the **8th character** (e.g. "A") designates which **UserBar** definition to use. **UBHeader** (UBName = that character) and **UBDetail** define the barcode: each character position (UBDispLoc) is filled from a source field (UBField, UBFieldCharLoc, UBLength). Sources include Serial, Product, Goals (e.g. PackDate, KillDate, SerialNum, NetWgt, Lot, ShiftProdCodeCount from Totals). The result is a custom barcode string used for **BARCODE** / **HUMANREADABLE** or the specific USERBAR token.

---

## 13. Reprint and Label Editing UI

- **Reprint**: **system/reprint-case-lbl.w** allows reprinting case labels. It uses **tt-Reprint-Controls.LabelFile** (or default) and builds **vLabelFile = tt-Reprint-Controls.LabelFile + '.lbl'** (or **'labels\' + vLabelFile**). **RUN PrintLabel in gh_PrintLabel** is called with the serial/case data and the chosen label file.
- **Label file selection**: In **weigh.w**, **fiLabelFile** displays the product label file (with **.lbl** appended on leave). If **Product.LabelFile** is blank or ?, a message (**PrintLabels.Msg.MsgProdLabelFile**) can prompt for a label file. **v-tt-product.w** (system product maintenance) shows **fiLabelFile** and can set it from **Goals.LabelFile** when editing under a goal.
- **Edit labels**: **sobjects/s-editlabels.w** lets users create/copy label files; it enforces **.lbl** extension and 8-character-max base name before **.lbl**. **s-setupprinter.w** validates that the selected file has extension **.lbl** when setting up the label printer.

---

## 14. Business Rules Summary

- **Label file**: Must exist at path returned by **SEARCH(piLabelFile)**; otherwise **PrintLblInits** returns ? and **PrintLabel** exits without printing and shows "Label File Not Found".
- **Label weight**: Computed by **Set-Label-Wgt** from **Product.PackType** and Min/Max/Std and total weight; stored on **Serial.LabelWgt** and used for **LABELWEIGHT** and barcode formatting.
- **Token source**: Each token has a defined source (CaseLabelSource, Product, Goals, UserBarString, CrossRef, session/site config). Unknown tokens are not substituted or are blank.
- **Repetitions**: **ProductProcess.Repetitions** controls how many times the same label is printed for one case (e.g. 2 labels per case).
- **Device**: The step’s **Device** (DeviceID) and the COM handle (from open printer) must be valid for **WriteToDevice** to succeed. **gPrinterAttached** must be true to actually send to device.

---

## 15. Redevelopment Notes

- Replicate **label file** resolution: **labels/** + base name + **.lbl** (and optional .1st, .2nd, .can); **SEARCH** or equivalent to resolve path; fail cleanly if file missing.
- Replicate **tt-CaseLabelSource** (Serial-like + CaseSeq, MarkDownPrice, PkgDesc3) and fill from Serial and Goals before calling the label renderer.
- Implement **token substitution** with the same token names and sources (Serial/case, Product, Goals, UserBarString NVP, CrossRef, config). Support optional "TableName." prefix and override suffix (e.g. **-0**).
- **Set-Label-Wgt**: Implement PackType rules (C/F/U/V/H) for label weight; use that value for **Serial.LabelWgt** and for **LABELWEIGHT** / barcode.
- **PrintLabel** equivalent: Read template (binary-safe), substitute tokens, send result to **WriteToDevice** with device ID and handle. Support sample and cancel modes.
- **Userbar**: Replicate UBHeader/UBDetail and the USERBARA–Z token logic (character-by-character build from configured source fields).
- **Product.LabelFile**, **Item.LabelFile**, **Goals.LabelFile**, **ProductProcess.OutputLabel** and **Device** must drive which template and which printer are used for each print.
