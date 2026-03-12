---
Risk-Level: P0
Business-Rule-Id: BR-ProductProcess-003
Deprecated: false
---

# Description

ValidProcess table defines available process types (Input, Output, and Group). ProcessID is the primary key and must be unique. ProductProcess.ProcessID is a foreign key to ValidProcess.ProcessID. Each ProductProcess step references one ValidProcess.

# Source

- [[../Data Models/0.Legacy Database Schema]] — ValidProcess (ProcessID PK), ProductProcess (ProcessID FK)
- [[../Data Models/Product-Process/Product-Process legacy]] — ProcessID must exist in ValidProcess
- slc.df ValidProcess "Input, Output, and 'Group'"

# Impacted Systems

ValidProcess table, ProductProcess, weigh flow (process steps), validateprodprocess.p.

# Traceability

- ValidProcess: ProcessID, Description, DefaultProcess; index ProcessID unique
- ProductProcess.ProcessID X(20) references ValidProcess

# Assertions

- ProcessID unique in ValidProcess.
- Every ProductProcess.ProcessID must exist in ValidProcess (validation error "Invalid ProcessID" if missing).
- ValidProcess entries define the set of valid process types (e.g. CasePrint, CaseWeight, ItemPrint, PalletOutput).

# Related Information

