---
Risk-Level: P0
Business-Rule-Id: BR-ProductProcess-005
Deprecated: false
---

# Description

When TT-ProductProcess.OutputLabel is not ? and not "", the label file must exist. validateprodprocess.p Procedure ValidateLabels: file-info:file-name = 'labels/' + TT-ProductProcess.OutputLabel + '.lbl'. If file-info:pathname = ?, run dialogs/d-tsmsgbox.w with message "Label File (.lbl): [path] | [ProcessSequence]" not found (PrintLabels.EMsg.FileMissing) and set poOK = no. If OutputLabel is ? or "", validation is skipped (RETURN without check). Per spec 2122-11 and RM 2858, blank labels are not validated.

# Source

- progress-SLC/print/validateprodprocess.p — Procedure ValidateLabels: IF TT-ProductProcess.OutputLabel = ? OR TT-ProductProcess.OutputLabel = "" THEN RETURN; else check file-info:pathname for 'labels/' + OutputLabel + '.lbl'

# Impacted Systems

ProductProcess, OutputLabel, labels/ directory, print/validateprodprocess.p, s-printlbl.w.

# Traceability

- [[../Drafts/Product/BR-Product-005-Label-file-must-exist-for-PrintLabel]]
- [[../Data Models/Labels/Labels - legacy]] — label path resolution

# Assertions

- When OutputLabel is non-null and non-blank, file 'labels/' + OutputLabel + '.lbl' must exist.
- When file is not found, poOK = no and user is shown FileMissing message.
- When OutputLabel is ? or "", no file existence check is performed.

# Related Information

