---
Risk-Level: P0
Business-Rule-Id: BR-Totals-002
Deprecated: false
---

# Description

When lib/updatetotals.p cannot obtain an exclusive lock on the Totals record within vMaxTimeToWaitMS (15000 ms, 15 seconds), it leaves the 050-Process-Serial block without updating Totals or Goals and sets v-RetVal (po-RetVal) to "No totals update". The caller should check return-value after Run UpdateTotals; a return-value of "" means no errors; "No totals update" means the update was skipped due to lock timeout.

# Source

- progress-SLC/lib/updatetotals.p — 125-find-totals loop: if vElapsedTime > vMaxTimeToWaitMS then RUN Logger(...), v-RetVal = "No totals update", leave 050-Process-Serial. vMaxTimeToWaitMS init 15000.

# Impacted Systems

Totals table, Goals table, lib/updatetotals.p, callers that run UpdateTotals (e.g. serial add/delete flow).

# Traceability

- [[../Drafts/Totals/BR-Totals-001-Shift-Scale-ProductCode-PdnDate-unique]]
- [[../Data Models/0.Legacy Database Schema]] — Totals

# Assertions

- If an exclusive lock on the matching Totals record cannot be obtained within 15 seconds, UpdateTotals does not perform the update and returns "No totals update".
- Caller must check return-value; empty string means success, non-empty means error or "No totals update".

# Related Information

