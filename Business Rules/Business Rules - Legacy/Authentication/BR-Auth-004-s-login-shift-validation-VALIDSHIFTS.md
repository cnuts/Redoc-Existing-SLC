---
Risk-Level: P0
Business-Rule-Id: BR-Auth-004
Deprecated: false
---

# Description

Before accepting login, s-login.w validates the shift. It finds ConfigOptionMaster where OptionName = "VALIDSHIFTS" no-lock. If ConfigOptionMaster is not available, or if LOOKUP(STRING(FILLSHIFT), ConfigOptionMaster.AcceptableValues, "|") = 0 (the entered shift is not in the pipe-delimited list of acceptable values), the login is rejected: po-Ok is set to no and a message is shown (get-Lang-Lbl "Login.Error" and a bad-shift message). For the backdoor (CFSUSER) and SecurityOverride paths, if the shift is invalid the code defaults fillshift to '1' and fillshift:screen-value to '1' instead of rejecting. Thus normal login requires the shift to be one of the values in ConfigOptionMaster.VALIDSHIFTS AcceptableValues; special paths can default to shift '1'.

# Source

- progress-SLC/sobjects/s-login.w: ip-Validate-User and ip-Validate-Backdoor, ip-Validate-Override. FIND ConfigOptionMaster WHERE OptionName = "VALIDSHIFTS". If not avail or lookup(String(FILLSHIFT), AcceptableValues, "|") = 0 then 450-bad-shift: po-Ok = no, message (or in backdoor/override: fillshift = '1').

# Impacted Systems

s-login.w, ConfigOptionMaster (VALIDSHIFTS), fillshift widget, gShift (set after successful login).

# Traceability

- AcceptableValues is pipe-delimited (e.g. "1|2|3"); FILLSHIFT is the widget value for shift selection.

# Assertions

- Login does not succeed if shift is not in VALIDSHIFTS unless using backdoor or SecurityOverride, in which case shift defaults to '1'.

# Related Information

- [[BR-Auth-001-Username-and-Password-required]]
- [[BR-Operator-004-Login-Operator-must-exist-password-MD5-or-legacy]]
