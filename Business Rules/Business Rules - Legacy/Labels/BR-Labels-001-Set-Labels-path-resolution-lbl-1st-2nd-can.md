---
Risk-Level: P0
Business-Rule-Id: BR-Labels-001
Deprecated: false
---

# Description

The Set-Labels procedure (print/set-labels.p) resolves the physical label file paths from a logical label name (ip-LabelFile). If ip-LabelFile is blank, "DEFAULT" is used. The base directory is "labels/". It returns: op-FirstLabel (main label file), op-QuickLabel (for subsequent labels in a run), op-CancelLabel (cancel label), and op-LabelFile (the logical name used). Resolution uses FILE-INFO: op-FirstLabel is set to the full path of "labels/" + TRIM(op-LabelFile) + ".lbl" if that file exists, else ""; op-QuickLabel is the path of the .1st file if it exists, then the .2nd file if it exists, otherwise falls back to op-FirstLabel; op-CancelLabel defaults to "labels/cancel.lbl" and is overridden to "labels/" + TRIM(op-LabelFile) + ".can" if that file exists.

# Source

- progress-SLC/print/set-labels.p: PROCEDURE Set-Labels; op-LabelFile = (if ip-LabelFile = "" then "DEFAULT" else ip-LabelFile); FILE-INFO:FILE-NAME = "labels/" + TRIM(op-LabelFile) + ".lbl" → op-FirstLabel; FILE-INFO for .1st then .2nd → op-QuickLabel; op-CancelLabel = "labels/cancel.lbl", FILE-INFO for .can overrides
- Invoked by case-label.p, case-print.p, case-family-print.p, case-batch-serial.p (per header comments)

# Impacted Systems

set-labels.p, labels/ directory, .lbl / .1st / .2nd / .can files, callers (case-label.p, case-print.p, etc.), tt-serial-source.i (output params).

# Traceability

- Logical name (e.g. from Product.LabelFile or ProductProcess.OutputLabel) is mapped to up to four physical files: main (.lbl), first/quick (.1st, .2nd), cancel (.can).
- When .1st or .2nd do not exist, op-QuickLabel equals op-FirstLabel so the same file is used for first and subsequent labels.

# Assertions

- op-FirstLabel is empty string if the .lbl file does not exist; callers should validate label existence before use (see [[BR-Product-005-Label-file-must-exist-for-PrintLabel]] and ProductProcess rules).
- File existence is checked via FILE-INFO:FILE-TYPE NE ?.

# Related Information

- [[BR-Labels-002-PrintLblInits-SEARCH-and-missing-label-message]]
- [[BR-Product-005-Label-file-must-exist-for-PrintLabel]]
