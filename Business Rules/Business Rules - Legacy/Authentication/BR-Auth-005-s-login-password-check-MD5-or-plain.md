---
Risk-Level: P0
Business-Rule-Id: BR-Auth-005
Deprecated: false
---

# Description

In s-login.w, after the Operator row is found, password verification depends on the length of Operator.Password. If LENGTH(Operator.Password) <> 32, the password is treated as plain text: the entered password (fi_PassWd) is compared directly to Operator.Password. If they are not equal, login fails: BELL twice, po-OK = FALSE, message get-Lang-Lbl "Login.BadPassword", fi_PassWd:SCREEN-VALUE is cleared, and the procedure returns. If LENGTH(Operator.Password) = 32, the password is treated as MD5 hex: the code computes vPWRaw = MESSAGE-DIGEST('MD5', fi_PassWd), vPWEncoded = HEX-ENCODE(vPWRaw), and compares vPWEncoded to Operator.Password. If they are not equal, the same failure handling applies (BELL, message, clear password field). Thus 32-character stored passwords are verified using MD5 hash of the entered password; non-32-character stored passwords are verified by plain comparison.

# Source

- progress-SLC/sobjects/s-login.w: ip-Validate-User and ip-AuthLocally. IF LENGTH(Operator.Password) <> 32 THEN IF (fi_PassWd <> Operator.Password) THEN fail. ELSE vPWRaw = MESSAGE-DIGEST('MD5', fi_PassWd), vPWEncoded = HEX-ENCODE(vPWRaw); IF (vPWEncoded <> Operator.Password) THEN fail.

# Impacted Systems

s-login.w, Operator.Password, fi_PassWd, login success/failure and message labels (Login.BadPassword, Login.Error).

# Traceability

- 32-character hex string is the expected format for MD5 (16 bytes encoded as 32 hex chars).

# Assertions

- No other hash algorithm is used in this login path; plain text is only used when stored length is not 32.

# Related Information

- [[BR-Auth-00002-Password-storage-SHA512-salt]]
- [[BR-Operator-004-Login-Operator-must-exist-password-MD5-or-legacy]]
