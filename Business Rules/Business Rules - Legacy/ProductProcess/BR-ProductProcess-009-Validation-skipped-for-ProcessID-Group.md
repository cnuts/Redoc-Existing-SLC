---
Risk-Level: P0
Business-Rule-Id: BR-ProductProcess-009
Deprecated: false
---

# Description

When validating product processes (print/weigh-checkprodprocess.p procedure CheckProdProcess), Device, ButtonImage, OutputLabel, and Routine checks are run only for TT-ProductProcess records where ProcessID is not 'Group'. The loop is "for each TT-ProductProcess where TT-ProductProcess.ProductCode = TT-Product.ProductCode and TT-ProductProcess.ProcessID <> 'Group'". So process records with ProcessID = 'Group' (group header records) are excluded from ValidateDevices, ValidateButtons, ValidateLabels, and ValidateRoutines. This avoids requiring a device, label, routine, or button image for group header rows that define repetition structure rather than executable steps.

# Source

- progress-SLC/print/weigh-checkprodprocess.p — procedure CheckProdProcess: for each TT-ProductProcess where ProductCode = TT-Product.ProductCode and TT-ProductProcess.ProcessID <> 'Group' no-lock: run ValidateDevices, ValidateButtons (if ButtonImage set), ValidateLabels (if InputProcess = false), ValidateRoutines

# Impacted Systems

print/weigh-checkprodprocess.p, print/validateprodprocess.p, TT-ProductProcess, ProcessID = 'Group'.

# Traceability

- [[BR-ProductProcess-004-Device-and-AlternateDevice-must-exist-validateprodprocess]] — Device validation applies only when ProcessID <> 'Group'
- [[BR-ProductProcess-005-OutputLabel-file-must-exist-when-not-blank]], [[BR-ProductProcess-006-Routine-file-must-exist-p-r-or-w]], [[BR-ProductProcess-007-ButtonImage-files-must-exist-when-set]] — same skip for Group records

# Assertions

- For each TT-ProductProcess with ProcessID <> 'Group', ValidateDevices, ValidateButtons (when ButtonImage set), ValidateLabels (when InputProcess = false), and ValidateRoutines are run.
- TT-ProductProcess records with ProcessID = 'Group' are not validated for Device, OutputLabel, Routine, or ButtonImage.

# Related Information

