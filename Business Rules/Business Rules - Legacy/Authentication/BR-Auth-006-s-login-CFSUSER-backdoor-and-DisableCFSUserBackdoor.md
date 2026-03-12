---
Risk-Level: P0
Business-Rule-Id: BR-Auth-006
Deprecated: false
---

# Description

s-login.w implements a backdoor login for operator id 'CFSUSER'. The backdoor password is the current time string (vTime). If the user enters operator = CFSUSER (compared to vBackdoorOperator, which is initialized to 'CFSUSER') and password = vTime, and the site config DisableCFSUserBackdoor is not true, login succeeds via ip-Validate-Backdoor. If v-DisableCFSUserBackdoor is true, or if the entered password (v-fill-passwd) does not equal vTime, the backdoor fails: BELL twice and the code goes to 550-Bad-PW (login rejected). After a successful backdoor, shift is still validated against VALIDSHIFTS; if invalid, fillshift defaults to '1'. Thus CFSUSER can log in with password equal to the current time unless DisableCFSUserBackdoor is set.

# Source

- progress-SLC/sobjects/s-login.w: vBackdoorOperator = 'CFSUSER'. ip-Validate-Backdoor: vTime set to current time string; if v-DisableCFSUserBackdoor or v-fill-passwd <> vTime then 550-Bad-PW. Else proceed; VALIDSHIFTS check with default fillshift = '1' if invalid.

# Impacted Systems

s-login.w, vBackdoorOperator, vTime, v-DisableCFSUserBackdoor (from site config or registry), CFSUSER operator.

# Traceability

- Backdoor is intended for support; DisableCFSUserBackdoor allows sites to disable it.

# Assertions

- Only operator id CFSUSER uses the time-as-password backdoor; other operators use normal Operator table and password check.

# Related Information

- [[BR-Operator-005-f-get-permission-CFSUSER-and-SecurityOverride-bypass]]
