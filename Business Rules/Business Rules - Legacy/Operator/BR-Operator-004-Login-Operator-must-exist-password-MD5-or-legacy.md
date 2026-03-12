---
Risk-Level: P0
Business-Rule-Id: BR-Operator-004
Deprecated: false
---

# Description

Login (sobjects/s-login.w) requires that the Operator record exist for the entered operator ID. FIND FIRST Operator WHERE Operator.Operator EQ ip-Operator (or fi_operator) NO-LOCK NO-ERROR; IF NOT AVAIL(Operator) then BELL, op-OK = FALSE, run d-tsmsgbox with get-Lang-Lbl "Login.NoOperator" + operator id, and return. Password validation: if LENGTH(Operator.Password) <> 32 then the password is treated as legacy plain text and compared directly to the entered password (fi_PassWd); if length is 32, the entered password is hashed with MESSAGE-DIGEST('MD5', ip-Password) and HEX-ENCODE; the result is compared to Operator.Password. If the password does not match, BELL, op-OK = FALSE, message get-Lang-Lbl "Login.BadPassword", clear the password field, and return. So operator must exist and password must match (legacy plain or MD5 hex) for login to succeed.

# Source

- progress-SLC/sobjects/s-login.w — procedure ip-AuthLocally and login flow: FIND Operator by ip-Operator; IF NOT AVAIL show Login.NoOperator. IF LENGTH(Operator.Password) <> 32 then IF fi_PassWd <> Operator.Password then error Login.BadPassword. ELSE vPWEncoded = HEX-ENCODE(MESSAGE-DIGEST('MD5', ip-Password)); IF vPWEncoded <> Operator.Password then error Login.BadPassword.

# Impacted Systems

Operator table (Operator, Password), sobjects/s-login.w, get-Lang-Lbl (Login.NoOperator, Login.BadPassword, Login.Error).

# Traceability

- [[BR-Operator-001-Permission-gate]] — permission check uses gOperator set at login
- [[BR-Auth-001-Username-and-Password-required]] — login requires credentials; Operator record is the stored identity

# Assertions

- Operator row must exist for the entered operator ID or login fails with Login.NoOperator.
- Password is validated as 32-char MD5 hex or legacy plain; mismatch shows Login.BadPassword and clears password field.

# Related Information

