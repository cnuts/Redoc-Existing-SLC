---
Risk-Level: P0
Business-Rule-Id: BR-Operator-005
Deprecated: false
---

# Description

Function f-get-permission (lib/services.p) returns TRUE (permission granted) when gOperator = 'CFSUSER' or when gSecurityOverride is true, without consulting OperatorAppFunction. Otherwise it finds the first OperatorAppFunction where OperatorAppFunction.ID = pi-AppFunction and OperatorAppFunction.Operator = gOperator no-lock no-error; if not available or OperatorAppFunction.Permitted is false, it returns FALSE; else it returns TRUE. So CFSUSER and the security override bypass all permission checks; all other operators are gated by the presence and Permitted flag of the corresponding OperatorAppFunction row.

# Source

- progress-SLC/lib/services.p — FUNCTION f-get-permission RETURNS LOGICAL (INPUT pi-AppFunction AS CHAR): if gOperator = 'CFSUSER' then return true. if gSecurityOverride then return true. find first OperatorAppFunction where ID = pi-AppFunction and Operator = gOperator no-lock no-error. if not avail or not OperatorAppFunction.Permitted then return false. else return true.

# Impacted Systems

lib/services.p, gOperator, gSecurityOverride, OperatorAppFunction table, all callers of f-get-permission (macros/f-get-permission.i, UI enable/disable).

# Traceability

- [[BR-Operator-001-Permission-gate]] — f-get-permission implements the gate; this rule documents the bypass cases
- [[BR-Operator-002-Operator-unique-AppFunction-mapping]] — OperatorAppFunction (Operator, ID)

# Assertions

- When gOperator is 'CFSUSER', f-get-permission always returns true.
- When gSecurityOverride is true, f-get-permission always returns true.
- Otherwise permission is true only if OperatorAppFunction exists for that Operator and ID and Permitted = true.

# Related Information

