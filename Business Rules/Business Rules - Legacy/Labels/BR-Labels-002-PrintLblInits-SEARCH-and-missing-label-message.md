---
Risk-Level: P0
Business-Rule-Id: BR-Labels-002
Deprecated: false
---

# Description

The PrintLblInits procedure in lib/printlabel.p resolves the label file path passed to PrintLabel. It accepts the label file name/path (ip-LabelFile) and returns the path to open (poPathNameToOpen) using SEARCH(ip-LabelFile). If SEARCH returns ?, the procedure runs Log(1, ip-LabelFile), displays a message via d-tsmsgbox with title "Label File Not Found" and text "lib/printlabel.p Printlblinits Missing Label OR folder was moved : Cancel last case", and returns without assigning the output (so it remains ?). The PrintLabel caller then checks chrPathNameToOpen and returns without printing if it is ?. Thus printing is aborted when the label file cannot be found at runtime.

# Source

- progress-SLC/lib/printlabel.p: PROCEDURE PrintLblInits (ip-AddOrDelete, ip-LabelFile, output poPathNameToOpen); poPathNameToOpen = SEARCH(ip-LabelFile) no-error; IF poPathNameToOpen EQ ? THEN RUN Log(1, ip-LabelFile), RUN dialogs/d-tsmsgbox.w "Label File Not Found", "Printlblinits Missing Label OR folder was moved : Cancel last case", RETURN. PrintLabel: RUN printLblInits input piLabelFile output chrPathNameToOpen; IF chrPathNameToOpen = ? THEN RETURN
- Spec 2122-11: PrtLblInits error msg when label is not found; should only appear if user is on print screen and label file was deleted after validation

# Impacted Systems

printlabel.p, PrintLabel procedure, Log (gh_LogActivity), d-tsmsgbox, PROPATH (SEARCH resolution).

# Traceability

- SEARCH uses the Progress PROPATH to locate the file; the label path may be relative or absolute.
- When the file is missing, the user is instructed to cancel the last case; no label is printed.

# Assertions

- PrintLabel does not open the label stream or print when chrPathNameToOpen is ?.
- The message is intended for the case where the label existed at screen entry but was deleted or moved before the print call (e.g. after validation).

# Related Information

- [[BR-Labels-001-Set-Labels-path-resolution-lbl-1st-2nd-can]]
- [[BR-Product-005-Label-file-must-exist-for-PrintLabel]]
