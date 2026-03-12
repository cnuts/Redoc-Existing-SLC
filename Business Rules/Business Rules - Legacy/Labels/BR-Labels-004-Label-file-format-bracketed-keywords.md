---
Risk-Level: P0
Business-Rule-Id: BR-Labels-004
Deprecated: false
---

# Description

Label files are text (or binary) files that are read by PrintLabel (lib/printlabel.p) and sent to the label printer. Each line is read and keywords in brackets (e.g. [netwgt], [serialnum]) are replaced with values from the database and context (Serial, Product, Goals, Modify, etc.). The reconstructed line is then sent to the printer. The procedure opens the file with INPUT STREAM s-Label FROM value(chrPathNameToOpen) BINARY UNBUFFERED NO-CONVERT and processes character by character (READKEY) to detect left bracket 91 and replace matching keyword strings. Bartender-created labels may have images encoded in the file and field names with a TableName prefix in brackets.

# Source

- progress-SLC/lib/printlabel.p: Comment "replace its keywords, which are in brackets, with meaningful data from the database"; INPUT STREAM s-Label FROM value(chrPathNameToOpen) BINARY UNBUFFERED NO-CONVERT; 100-reformat-lbl: READKEY, detect bracket 91, match and replace; parameters include Serial num, AddOrDel, labelfile.lbl, com-handle; buffers b-tt-Product, tt-ItemLabelSource, tt-CaseLabelSource, Goals
- History: Bartender labels (images encoded, Tablename before field in brackets); Product Text subfields ProductText21A–Z; various customer-specific fields (Type2 UPC-A, DEPTCODEDATE, etc.)

# Impacted Systems

printlabel.p, label .lbl/.1st/.2nd files, Serial/Product/Goals/Modify data, printer com-handle, CreateBarcode, UserBar, format procedures.

# Traceability

- Bracketed placeholders are replaced before output; unknown or null fields may be replaced with blank or null per product/text rules (e.g. text11–99 not considered bad field, return null).
- Label content is driven by Product, ProductProcess (OutputLabel), and Set-Labels resolution.

# Assertions

- The label file is consumed once per print; stream is opened from chrPathNameToOpen (resolved by PrintLblInits).
- Replacement logic supports many predefined keywords and product-specific text fields; see printlabel.p for the full list.

# Related Information

- [[BR-Labels-001-Set-Labels-path-resolution-lbl-1st-2nd-can]]
- [[BR-Labels-002-PrintLblInits-SEARCH-and-missing-label-message]]
- [[BR-Product-006-Set-Label-Wgt-PackType-logic-from-source]]
