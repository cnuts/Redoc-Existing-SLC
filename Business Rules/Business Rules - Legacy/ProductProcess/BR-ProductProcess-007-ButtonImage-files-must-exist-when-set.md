---
Risk-Level: P0
Business-Rule-Id: BR-ProductProcess-007
Deprecated: false
---

# Description

When TT-ProductProcess.ButtonImage is not ? and not "", two image files must exist: images/[ButtonImage]-u.bmp and images/[ButtonImage]-i.bmp. validateprodprocess.p Procedure ValidateButtons: file-info:file-name = 'images/' + TT-ProductProcess.ButtonImage + '-u.bmp'; if file-info:pathname = ? then show FileMissing message and poOK = no; same for '-i.bmp'. If either file is missing, validation fails with PrintLabels.EMsg.FileMissing ("[path] | [ProcessSequence]: Please update or add file.").

# Source

- progress-SLC/print/validateprodprocess.p — Procedure ValidateButtons: check images/ + ButtonImage + '-u.bmp' and '-i.bmp'; if either pathname = ? then d-tsmsgbox and poOK = no

# Impacted Systems

ProductProcess.ButtonImage, images/ directory, print/validateprodprocess.p, s-printlbl.w.

# Traceability

- [[../Data Models/0.Legacy Database Schema]] — ProductProcess ButtonImage x(10), .bmp assumed
- [[../Data Models/Product-Process/Product-Process legacy]]

# Assertions

- When ButtonImage is set, images/[ButtonImage]-u.bmp must exist.
- When ButtonImage is set, images/[ButtonImage]-i.bmp must exist.
- When either is missing, poOK = no and user is shown FileMissing message.

# Related Information

