---
Risk-Level: P1
Business-Rule-Id: BR-ProductProcess-012
Deprecated: false
---

# Description

utilities/ProductProcess-recon.p normalizes ProcessID string values in the ProductProcess table. For each ProductProcess where ProcessID <> "group", it replaces: "Case Output" with "CaseOutput"; "Pallet Output" with "PalletOutput"; "Container-PreTare" with "ContainerPreTare"; "Default Case Weight" with "DefaultCaseWeight"; "ContainerPre-Tare" with "ContainerPreTare"; "ContainerPreTareWeight" with "ContainerPreTare". These replacements align ProcessID with the expected token values used elsewhere (e.g. ValidProcess, default process names in weigh-create-tt-prodprocess.p). The utility is a reconciliation/data-cleanup procedure, not runtime validation.

# Source

- progress-SLC/utilities/ProductProcess-recon.p — for each productprocess where processid <> "group": replace "Case Output"→"CaseOutput", "Pallet Output"→"PalletOutput", "Container-PreTare"→"ContainerPreTare", "Default Case Weight"→"DefaultCaseWeight", "ContainerPre-Tare"→"ContainerPreTare", "ContainerPreTareWeight"→"ContainerPreTare"

# Impacted Systems

ProductProcess.ProcessID, utilities/ProductProcess-recon.p, ValidProcess.ProcessID (expected values).

# Traceability

- [[BR-ProductProcess-003-ValidProcess-ProcessID-unique]] — ProcessID must exist in ValidProcess; normalized names match ValidProcess entries
- [[BR-ProductProcess-008-ProductProcess-RI-when-ValidProcess-not-found]] — RI utility; recon is separate normalization utility

# Assertions

- ProductProcess-recon.p updates ProcessID in place with the listed string replacements.
- Records with ProcessID = "group" are not modified by this utility.

# Related Information

