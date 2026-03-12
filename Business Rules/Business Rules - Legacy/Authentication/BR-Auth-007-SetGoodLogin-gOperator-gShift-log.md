---
Risk-Level: P0
Business-Rule-Id: BR-Auth-007
Deprecated: false
---

# Description

After a successful login validation (ip-Validate-User, ip-AuthLocally, ip-Validate-Backdoor, or ip-Validate-Override), s-login.w runs SetGoodLogin. SetGoodLogin sets po-Ok = TRUE, gOperator = fi_operator (the entered operator id), and gShift = INT(fillShift). It then appends a line to the file "logs/logins.log" containing the date, time, "Oper: " + gOperator, "Shift: " + gShift, and Operator.FirstName and Operator.LastName. Thus every successful login records the operator, shift, and name to logs/logins.log; gOperator and gShift are the session globals used for the rest of the application.

# Source

- progress-SLC/sobjects/s-login.w: PROCEDURE SetGoodLogin OUTPUT po-OK. ASSIGN po-Ok = TRUE, gOperator = fi_operator, gShift = int(fillShift). OUTPUT TO value("logs/logins.log") APPEND. PUT string(today) " " string(time,"HH:MM:SS") " Oper: " gOperator " Shift: " gShift " " Operator.FirstName Operator.LastName skip. OUTPUT CLOSE.

# Impacted Systems

s-login.w, gOperator, gShift (gLoginVars.i), logs/logins.log, all code that uses gOperator or gShift.

# Traceability

- SetGoodLogin is called from ip-Validate-User, ip-AuthLocally, ip-Validate-Backdoor, ip-Validate-Override, and from ipTryLogin (login server path).

# Assertions

- gOperator and gShift are only set in SetGoodLogin after a successful login; the log file is append-only.

# Related Information

- [[BR-Auth-001-Username-and-Password-required]]
