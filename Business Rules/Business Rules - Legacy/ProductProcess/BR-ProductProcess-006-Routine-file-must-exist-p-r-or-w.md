---
Risk-Level: P0
Business-Rule-Id: BR-ProductProcess-006
Deprecated: false
---

# Description

When validating ProductProcess, the Routine (when set) must exist as a file in the print\ directory with extension .p, .r, or .w. validateprodprocess.p Procedure ValidateRoutines: file-info:file-name = 'print\' + TT-ProductProcess.Routine + '.p'; if not found try .r; if not found try .w. If all three are missing (file-info:pathname = ?), run dialogs/d-tsmsgbox.w with message "Process Routine (.p/.r): [path] | [ProcessSequence]" not found and set poOK = no.

# Source

- progress-SLC/print/validateprodprocess.p — Procedure ValidateRoutines: try print\ + Routine + .p, then .r, then .w; if file-info:pathname = ? for .w then message and poOK = no

# Impacted Systems

ProductProcess.Routine, print/ directory, print/validateprodprocess.p, s-printlbl.w.

# Traceability

- [[../Data Models/Product-Process/Product-Process legacy]] — Routine for Input Capture or Output Formatting; "If InputProcess=YES, Routine must be defined"
- BR-ProductProcess-001

# Assertions

- When Routine is set, at least one of print\Routine.p, print\Routine.r, or print\Routine.w must exist.
- When none exist, poOK = no and user is shown FileMissing message for Process Routine.

# Related Information

