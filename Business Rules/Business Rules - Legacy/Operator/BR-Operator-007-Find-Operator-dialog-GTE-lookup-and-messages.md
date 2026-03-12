---
Risk-Level: P1
Business-Rule-Id: BR-Operator-007
Deprecated: false
---

# Description

The Find Operator dialog (dialogs/d-operatorfind.w) sets gOperatorRowid from an Operator lookup by the entered operator ID (fiOperator). Procedure ip-find-Operator: FIND first Operator where Operator.Operator >= fiOperator no-lock no-error. If a row is found but Operator.Operator is not equal to fiOperator, the user is shown "Operator [fiOperator] Not Found, Found Next". If no row is found, the procedure finds the last Operator. If that exists, the user is shown "Operator [fiOperator] Not Found, Found Previous"; otherwise "No Operators Exist". In all cases where an Operator buffer is available, gOperatorRowid is set to rowid(Operator), and poCancel = false. So the dialog uses a greater-than-or-equal lookup; when the exact ID is missing, it positions to the next higher operator or the last operator and informs the user.

# Source

- progress-SLC/dialogs/d-operatorfind.w — procedure ip-find-Operator: assign fiOperator. FIND first Operator where Operator.Operator >= fiOperator no-lock no-error. if avail and Operator <> fiOperator: run d-tsmsgbox "Operator [fiOperator] Not Found, Found Next". if not avail: FIND last Operator no-lock no-error; if avail then "Found Previous" else "No Operators Exist". gOperatorRowid = rowid(Operator). poCancel = false.

# Impacted Systems

dialogs/d-operatorfind.w, Operator table, gOperatorRowid.

# Traceability

- [[BR-Operator-002-Operator-unique-AppFunction-mapping]] — Operator unique
- Find Operator sets global rowid for operator browser/selection

# Assertions

- Lookup is first Operator where Operator >= entered id.
- If exact match not found but a higher operator exists, user is told "Not Found, Found Next".
- If no operator >= id exists, last operator is used and user is told "Found Previous" or "No Operators Exist".
- gOperatorRowid is set to the found (or last) operator's rowid.

# Related Information

