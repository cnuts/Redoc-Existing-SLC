# UserBar Concept and Usage in the SLC Application

This document describes the **UserBar** concept in full: the **UBHeader** and **UBDetail** tables that define custom barcode/string formats, the **UBExamples** table used for setup guidance, how **lib/userbar.p** builds the userbar string from Serial/Product/Goals/Modify/Totals/CrossRef, how it ties into **Serial.UserBarString** and label printing, and how UserBar relates to Label, Serial, Product, Goals, and other concepts.

---

## 1. Overview

A **UserBar** is a **user-defined barcode or string** built character-by-character from configurable source fields. The application supports up to **26** named userbars, identified by a single letter **A** through **Z**. On a label template, the tokens **USERBARA**, **USERBARB**, … **USERBARZ** instruct **printlabel.p** to run **lib/userbar.p** with that letter; **userbar.p** finds the **UBHeader** row whose **UBName** equals the letter, then iterates over **UBDetail** rows for that name and assembles one character per detail from the configured **UBField** (e.g. SERIAL, PACK_DATE, LOT, ShiftProdCodeCount). The result is returned and substituted for the token. So UserBar allows site-specific barcode formats without code changes.

**Serial.UserBarString** (and **ItemSerial.UserBarString**) store **NVP** (name-value pair) data, including the **Barcode** value produced at serial creation (from **CreateBarCode**). That **Barcode** is what the **BARCODE** and **HUMANREADABLE** tokens use on the label. The **USERBARA–Z** tokens, by contrast, build a **separate** string at print time from **UBHeader/UBDetail**; that string can be an alternative barcode, a custom format for a scanner, or any fixed-length string derived from the same case/serial data.

---

## 2. UBHeader Table

**Source**: `slc.df`, table **UBHeader** (LABEL "UBHeader", DUMP-NAME "ubheader")

| Field | Type | Description |
|-------|------|-------------|
| **UBName** | character | x(20). **Single letter (A–Z)** that identifies this userbar. Primary key. Must match the 8th character of the token (e.g. USERBARA → "A"). |
| **UBLength** | integer | Total length of the userbar string (number of characters). **UBDetail** rows should have one row per position (UBDispLoc 1 to UBLength). |
| **UBAddDate** | date | When the userbar was added. |
| **UBAddOper** | character | Operator who added it. |

**Index**: **UBHIdx** (unique, primary) on **UBName**.

**Role**: **UBHeader** is the master record for one userbar definition. **UBName** is the letter passed from printlabel (SUBSTRING(chrStringToMatch, 8, 1)). **UBLength** defines how many **UBDetail** positions there are; the runtime builds the string by processing each **UBDetail** in **UBDispLoc** order and appending one character per row.

---

## 3. UBDetail Table

**Source**: `slc.df`, table **UBDetail** (LABEL "UBDetail", DUMP-NAME "ubdetail")

| Field | Type | Description |
|-------|------|-------------|
| **UBName** | character | x(8). Links to **UBHeader.UBName** (which userbar this detail belongs to). Part of primary key. |
| **UBDispLoc** | integer | Display position (1-based) in the userbar string. Part of primary key. Details are processed in **break by UBDetail.UBDispLoc** order. |
| **UBField** | character | x(15). **Source field name** (e.g. SERIAL, PACK_DATE, LOT, ShiftProdCodeCount, LITERAL). **userbar.p** has a **CASE CAPS(UBField)** that maps each name to a value from Serial, Product, Modify, Goals, Totals, CrossRef, or literals. |
| **UBFieldCharLoc** | integer | **Character position** within the source value (1-based). The character at this position is taken (e.g. SUBSTRING(sourceValue, UBFieldCharLoc, 1)). For fixed single chars (e.g. LITERAL), used as the character code or literal index. |
| **UBLength** | integer | Used for checksum calculations (e.g. CHECKSUM13, CHECKSUM31); otherwise the string is built one character per detail. |

**Index**: **UBDetIdx** (unique, primary) on **UBName**, **UBDispLoc**.

**Role**: Each **UBDetail** row contributes **one character** to the userbar. The value for that character comes from the **UBField** logic; **UBFieldCharLoc** (and sometimes **UBLength**) specify which character or range. So to show a 4-digit count, you add four **UBDetail** rows with the same **UBField** (e.g. ShiftProdCodeCount) and **UBDispLoc** 1, 2, 3, 4 and **UBFieldCharLoc** 1, 2, 3, 4.

---

## 4. UBExamples Table

**Source**: `slc.df`, table **UBExamples** (LABEL "UBExamples", DUMP-NAME "ubexampl")

| Field | Type | Description |
|-------|------|-------------|
| **UBField** | character | x(8). Matches **UBDetail.UBField** (the source field name). Primary key. |
| **UBExample** | character | x(48). Example value or format (e.g. "YYYYMMDD" for Packdate, "0123456789" for Literal). Used in the UI to guide setup; **not used by userbar.p** at runtime. |

**Index**: **UBExampIdx** (unique, primary) on **UBField**.

**Role**: Documentation/examples for each **UBField** so translators or admins know what values to expect. Records are seeded in **lib/add-to-runtime-db.p** (e.g. **7500-Build-UBExamples** for EXTENDEDPRICE, and historically TimeLot, ShiftProdCodeCount). There is no UI to add UBExamples; they are created in code.

---

## 5. userbar.p — Building the UserBar String

**Source**: `lib/userbar.p`

**Invocation**: Called from **printlabel.p** when a token **USERBARA** … **USERBARZ** is found. **RUN lib/userbar.p(INPUT SUBSTRING(chrStringToMatch, 8, 1), BUFFER pb-ItemLabelSource, BUFFER pb-CaseLabelSource, BUFFER pb-Goals, BUFFER b-tt-Product, OUTPUT op-Userbarstr)**. So the first argument is the single letter (e.g. "A"); the buffers supply the current case/serial and product/goals; the output is the built string.

**Parameters**:
- **ip-labelname**: Single character (A–Z), same as **UBHeader.UBName**.
- **pb-ItemLabelSource**: tt-ItemLabelSource (for item labels; used when UBField needs item data).
- **pb-CaseLabelSource**: tt-CaseLabelSource (like Serial; main source for serial, dates, weights, lot, order, etc.).
- **pb-Goals**: Goals buffer (for goal-based data when available).
- **b-tt-Product**: tt-Product (product descriptions, pack type, weights, text fields).
- **v-userbar** (output): The assembled string.

**Logic**:
1. **FIND UBHeader WHERE UBHeader.UBName = ip-labelname**. If not available, log and return (blank string).
2. **FOR EACH UBDetail WHERE UBDetail.UBName = UBHeader.UBName** ordered by **UBDispLoc**.
3. For each detail, **CASE CAPS(UBField)** to compute **v-Char** (one character):
   - **ShiftProdCodeCount**: From **Totals.LocalCount** (Totals for current shift, scale, product, PackDate from CaseLabelSource); **SUBSTRING(STRING(Totals.LocalCount,"9999"), UBFieldCharLoc, 1)**; else "X".
   - **PACK_TYPE**, **PACK_DATE**, **KILLDATE**, **LABEL_WEIGHT**, **LOT**, **NET_WEIGHT**, **ORDER**, **PACK_TIME**, **PRICE**, **PRODUCT**, **SCALE**, **SERIAL**, **SHIFT**: From **pb-CaseLabelSource** or **tt-product**; value converted to string, then **SUBSTRING(..., UBFieldCharLoc, 1)**.
   - **LONG_DESC1/2**, **SHORT_DESC1/2/3**, **TEXT01**…**TEXT10**: From **tt-product**; **SUBSTRING(field, UBFieldCharLoc, 1)**.
   - **BOXTARE**, **CUSTOMER**, **ExtendedPrice**, **MAX_WEIGHT**, **MIN_WEIGHT**, **STD_WEIGHT**, **TARE_WEIGHT**, **VOFFSET**, **WT_UNITS**, **XWT_UNITS**, **XLABEL_WEIGHT**: From **pb-CaseLabelSource**, **tt-product**, or **tt-modify**; same substring pattern.
   - **ADJUST**: PackDate − PrintDate formatted; **UBFieldCharLoc** picks character.
   - **Tare_Number**: From **tt-modify.TareToUse**.
   - **Wt_Unit_Num**: "2" for LB, "1" otherwise from **pb-CaseLabelSource.WgtUnits**.
   - **MANUF_ID**: From **CrossRef** "Product-MFGID" for product, or **gMfgID**; substring by **UBFieldCharLoc**.
   - **PLANT**: From **gPlantID**.
   - **UNIQUE**: From **f-get-session-parm("c","UNIQUENUM")**; substring by **UBFieldCharLoc**.
   - **TimeLot**: **CrossRef** Application **'TimeLot'**, ID ≥ PrintTime; **Descr** substring.
   - **LITERAL**: **v-Char = STRING(UBDetail.UBFieldCharLoc, "9")** (digit 1–9).
   - **ASCII**: **v-Char = CHR(UBDetail.UBFieldCharLoc)**.
   - **CHECKSUM13** / **CHECKSUM31**: Placeholder "E"; position and length stored; after the loop, **checksumvalue** procedure computes check digit and overwrites that position in **v-userbar** (mod 10, alternating 1/3 weights).
   - **DateAI**, **DateAIDate**: From **f-NVP-get-value(pb-CaseLabelSource.UserBarString, "DateAI" / "DateAIDate")**; date formatted via **f-get-dateformat** for DateAIDate.
4. **v-userbar = v-userbar + v-char** each iteration; **v-currentpos** incremented.
5. If **CHECKSUM13** or **CHECKSUM31** was used, **checksumvalue** is run and the result is written into the placeholder position in **v-userbar**.

**Dependencies**: **userbar.p** uses **tt-product**, **tt-modify** (via find-tt-product.i, find-tt-modify.i), **tt-Item**, **tt-ItemModify** (for item context), **Totals** (Shift, Scale, ProductCode, PdnDate from CaseLabelSource), **CrossRef** (Product-MFGID, TimeLot), **f-get-session-parm** (UNIQUENUM), **f-get-dateformat** (YYYYMMDD), **f-NVP-get-value** (UserBarString for DateAI/DateAIDate), and conversion factors **LBtoKGmultFactor** / **KGtoLBmultFactor** for **XLABEL_WEIGHT**.

---

## 6. Serial.UserBarString and Barcode Storage

**Serial.UserBarString** (and **ItemSerial.UserBarString**) store an **NVP** (name-value pair) string. At **serial creation** (**lib/createserial.p**):

- **CreateBarCode** (in **gh_CreateBarCode**) is run with serial number, product code, dates, weight, etc., and returns **chrReturnBarcode**.
- **pb-serial.UserBarString** is first set from **v-SerialNVPs** (other NVPs already collected), then **f-NVP-set-value** is used to add: **DateAI**, **DateAIDate**, **DeptCodeDate**, **DeptNumber**, **Counter**, **ScaleIndicator**, **VerificationEnabled**, **Barcode** (chrReturnBarcode), **SellByDate** (from Goals when present), and optionally Zulu/modify NVPs from **b-TT-Modify.ModText1**.
- So the **Barcode** in **UserBarString** is the **CreateBarCode** result. **printlabel.p** uses **f-NVP-get-value(pb-CaseLabelSource.UserBarString, "Barcode")** for the **BARCODE** and **HUMANREADABLE** tokens.

**USERBARA–Z** do **not** read this stored barcode; they **recompute** a string from **UBHeader/UBDetail** at print time. So you can have:
- **BARCODE** / **HUMANREADABLE**: The standard barcode stored at serial creation.
- **USERBARA** … **USERBARZ**: Custom formats (e.g. customer-specific barcode, or ShiftProdCodeCount string) defined by UBHeader/UBDetail.

Other keys stored in **UserBarString** and used elsewhere include **TotalUpdated**, **CancelDateTime**, **Verified**, **BatchID**, **ProcessOrder**, **PackOrder**, and goal/override data; **userbar.p** only reads **DateAI** and **DateAIDate** from it for those UBField cases.

---

## 7. Label and Print Flow

- **Label template**: Designer places **[USERBARA]** or **[USERBARB]**, …, **[USERBARZ]** where the custom string should appear.
- **printlabel.p**: When it finds such a token, it takes the 8th character (e.g. "A"), runs **lib/userbar.p** with that letter and the current **pb-CaseLabelSource**, **pb-Goals**, **b-tt-Product**, **pb-ItemLabelSource**; **op-Userbarstr** is assigned to **chrReplacementText** and substituted into the label.
- **userbar.w** (setup/test): Runs **lib/userbar.p** with a selected userbar name (e.g. from **gv-modvar**), then opens **labels/userbar.lbl**, replaces **[userbar]** with the built string, and sends the result to the printer (**WriteToDevice**). So operators can print a sample userbar without printing a full case label.

---

## 8. UI and Maintenance

- **viewers/v-ubhead.w**: SmartViewer over **UBHeader**. Displays/edits **UBName**, **UBLength**; can create/copy/delete userbar headers. When **UBLength** is set, **UBDetail** rows can be created (one per position). References **userbar.w** for the main setup flow.
- **print/userbar.w**: Main UserBar setup window. Populates **tt-product**, **tt-modify**, **tt-item**, **tt-item-modify** (and Serial-like data) so that **lib/userbar.p** has defaults for a test print. Calls **lib/userbar.p** with **entry(1, gv-modvar)** (selected userbar ID), then substitutes the result into **labels/userbar.lbl** and prints. Validates that **labels/userbar.lbl** exists. **s-cfgmenu.w** runs **print/userbar.w** (e.g. "User Bar" button).
- **queries/q-ubhead.w**: Query/browse over UBHeader.
- **dbclear**: Can delete **UBDetail** and **UBHeader** (delUBDetail, delUBHeader).

---

## 9. Relationships to Other Concepts

| Concept | Relationship to UserBar |
|--------|--------------------------|
| **Label** | **USERBARA–Z** are label tokens; **printlabel.p** calls **userbar.p** and substitutes the result. **BARCODE**/ **HUMANREADABLE** use **UserBarString** NVP "Barcode" (set at serial creation), not UBHeader/UBDetail. |
| **Serial** | **Serial.UserBarString** holds NVP data including **Barcode** (CreateBarCode result), **DateAI**, **DateAIDate**, etc. **pb-CaseLabelSource** (tt-CaseLabelSource, Serial-like) is the main input to **userbar.p** for SERIAL, PACK_DATE, LOT, NET_WEIGHT, etc. |
| **Product** | **tt-Product** (and **b-tt-Product**) supply PackType, descriptions, weights, Text1–10, WgtUnits for UBField cases. **CrossRef** "Product-MFGID" overrides MFGID per product for **MANUF_ID**. |
| **Goals** | **pb-Goals** is passed to **userbar.p**; **SellByDate** is stored in **UserBarString** in createserial when Goals has SellByDate. Goal-level data can influence userbar if UBField logic is extended. |
| **Modify** | **tt-Modify** supplies TareToUse, CustName, Lot, VarOffset, etc. for UBField (Tare_Number, CUSTOMER, LOT, VOFFSET). |
| **Totals** | **ShiftProdCodeCount** in userbar.p comes from **Totals.LocalCount** (Totals for current shift, scale, product, PackDate). |
| **CrossRef** | **Product-MFGID** (product-specific MFGID), **TimeLot** (time-based lot character from Descr). |
| **ItemSerial** | **ItemSerial.UserBarString** exists; **pb-ItemLabelSource** is passed to **userbar.p** for item-label context when needed. |
| **Config / Session** | **gMfgID**, **gPlantID**, **gShift**, **gScaleID**, **gProductCode**; **f-get-session-parm("c","UNIQUENUM")** for **UNIQUE**; **f-NVP-get-value(UserBarString, ...)** for DateAI/DateAIDate. |

---

## 10. Business Rules Summary

- **UBHeader.UBName** must be a single letter (A–Z) to match **USERBARA**…**USERBARZ** (8th character). **UBLength** must match the number of **UBDetail** rows (one per character position).
- **UBDetail**: One row per character position (UBDispLoc 1..UBLength); **UBField** must be a supported name in **userbar.p** (else no character or default). **UBFieldCharLoc** is the 1-based index into the source value string.
- **USERBARA–Z** are built at **print time** from current case/product/goals/modify/totals; they are independent of the **Barcode** stored in **UserBarString** at serial creation.
- **BARCODE** / **HUMANREADABLE** use **UserBarString** NVP **"Barcode"**, set in **createserial.p** from **CreateBarCode**.
- **UBExamples** are for documentation only; they are seeded in **add-to-runtime-db.p** and have no UI for end-user add. New **UBField** values require code in **userbar.p** (and optionally UBExamples) to take effect.

---

## 11. Redevelopment Notes

- Implement **UBHeader** (UBName, UBLength, UBAddDate, UBAddOper) and **UBDetail** (UBName, UBDispLoc, UBField, UBFieldCharLoc, UBLength); **UBExamples** (UBField, UBExample) optional for setup help.
- Implement **userbar.p** equivalent: given letter (A–Z), find UBHeader by UBName; for each UBDetail by UBName order by UBDispLoc, compute one character from UBField (and UBFieldCharLoc/UBLength) using CaseLabelSource, Product, Modify, Goals, Totals, CrossRef, session/config; support CHECKSUM13/CHECKSUM31; return concatenated string.
- **Serial.UserBarString** (and ItemSerial): Store NVP string; set **Barcode** at serial creation from your CreateBarCode equivalent; set DateAI, DateAIDate, and other NVPs as in createserial.p. **printlabel** (or equivalent) should resolve **BARCODE**/ **HUMANREADABLE** from **UserBarString** and **USERBARA–Z** by calling the userbar builder with the letter and current buffers.
- **userbar.w** flow: Select userbar, run userbar builder with default/sample data, substitute into **labels/userbar.lbl**, send to device. Ensure **labels/userbar.lbl** exists for test print.
- When adding new **UBField** names, add a CASE in the userbar builder and optionally seed **UBExamples**.
