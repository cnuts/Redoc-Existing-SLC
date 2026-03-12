---
Risk-Level: P0
Business-Rule-Id: BR-Operator-001
Deprecated: false
---

# Description

When a user attempts to perform a function, the system checks OperatorAppFunction for that Operator and the function ID (AppFunction.ID). If Permitted = YES, the function is allowed; otherwise the function is disabled or an error is shown. Only functions where Permitted = YES are enabled in the UI.

# Source

- [[../Data Models/Operator/Operator - legacy]] — §4 Operator and Permissions System, Permission Gate Logic
- [[../Data Models/Operator/Appfunction-Operator-OperatorAppfunction - legacy]]
- [[../Data Models/0.Legacy Database Schema]] — OperatorAppFunction (Operator, ID, Permitted)

# Impacted Systems

All screens and actions gated by AppFunction; OperatorAppFunction table; session permission set (g-OperatorPermissions).

# Traceability

- OperatorAppFunction: Operator FK, ID FK to AppFunction, Permitted logical
- Modify screen: Print.Modify.* permissions control which fields are editable

# Assertions

- Permission check: FIND OperatorAppFunction WHERE Operator = gOperator AND ID = v-FunctionID AND Permitted = YES.
- If not available, function is disabled or "You do not have permission" message shown.
- Serial.Operator and ItemSerial.Operator capture current user for audit trail.

# Related Information

