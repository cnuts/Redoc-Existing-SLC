---
Risk-Level: P0
Business-Rule-Id: BR-UserBar-008
Deprecated: false
---

# Description

The UserBar test print from print/userbar.w (button "Print User Bar") requires that the file labels\userbar.lbl exist. If file-info:file-name = 'labels\userbar.lbl' and FILE-INFO:FULL-PATHNAME = ?, the window runs dialogs/d-tsmsgbox.w with title "Print Sample User Bar" and message "labels\userbar.lbl not found, unable to print", then returns without printing.

# Source

- progress-SLC/print/userbar.w — ON CHOOSE OF bu-Print: file-info:file-name = 'labels\userbar.lbl'. if FILE-INFO:FULL-PATHNAME = ? then do: run dialogs/d-tsmsgbox.w (input 'Print Sample User Bar', input 'labels\userbar.lbl not found, unable to print'). return no-apply. end.

# Impacted Systems

print/userbar.w, labels/userbar.lbl, UserBar setup/test flow.

# Traceability

- [[../Data Models/UserBar/UserBar - legacy]] — §7 "userbar.w … validates that labels/userbar.lbl exists"
- [[../Drafts/ProductProcess/BR-ProductProcess-005-OutputLabel-file-must-exist-when-not-blank]]

# Assertions

- For UserBar test print to proceed, labels\userbar.lbl must exist.
- When the file is not found, the user is shown the message and the print action is aborted (return no-apply).

# Related Information

