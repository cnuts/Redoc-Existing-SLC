---
Risk-Level: P0
Business-Rule-Id: BR-ProductProcess-008
Deprecated: false
---

# Description

utilities/ProductProcess-RI.p: For each ProductProcess, FIND first ValidProcess where ValidProcess.ProcessID = ProductProcess.ProcessID no-lock no-error. If not avail ValidProcess, the program does not return an error "Invalid ProcessID". Instead it either (1) if ProductProcess.ProcessID = 'Defaultcaseweight', assign ProductProcess.ProcessID = 'CaseWeight' and display ProductCode, ProcessSequence, ProcessID; or (2) else UPDATE ProductProcess.ProcessID. So when ValidProcess is not found, the application corrects the ProcessID (Defaultcaseweight to CaseWeight) or updates the ProductProcess record rather than failing with an error.

# Source

- progress-SLC/utilities/ProductProcess-RI.p — for each productprocess: find first validProcess where ValidProcess.ProcessID = ProductProcess.ProcessID; if not avail ValidProcess then if ProcessID = 'Defaultcaseweight' then ProductProcess.ProcessID = 'CaseWeight' else update ProductProcess.ProcessID

# Impacted Systems

ProductProcess table, ValidProcess table, utilities/ProductProcess-RI.p (reconciliation utility).

# Traceability

- [[../Drafts/ProductProcess/BR-ProductProcess-001-ProcessID-Device-FK]] — vault doc states "error Invalid ProcessID"; code implements reconciliation instead
- ProductProcess-RI.p is a utility; validateprodprocess.p validates Device/Label/Routine/Buttons at runtime

# Assertions

- ProductProcess-RI.p: When ValidProcess is not found for a ProductProcess.ProcessID, the program updates ProcessID (Defaultcaseweight → CaseWeight) or performs UPDATE ProductProcess.ProcessID; it does not report "Invalid ProcessID" to the user.
- This is reconciliation/fix behavior, not runtime validation (runtime validation of Device etc. is in validateprodprocess.p).

# Related Information

