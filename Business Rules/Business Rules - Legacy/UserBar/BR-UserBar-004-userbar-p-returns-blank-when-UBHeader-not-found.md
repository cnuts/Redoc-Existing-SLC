---
Risk-Level: P0
Business-Rule-Id: BR-UserBar-004
Deprecated: false
---

# Description

When lib/userbar.p is called with ip-labelname (single letter A–Z) and no UBHeader row exists with UBHeader.UBName = ip-labelname, the procedure does not raise an error. It runs Log in gh_LogActivity (9, ip-labelName) and returns a blank string (v-userbar is never assigned in that branch, so the output remains ""). The label token (e.g. USERBARA) is therefore replaced with nothing.

# Source

- progress-SLC/lib/userbar.p — Find UBHeader where UBHeader.UBName = ip-labelname no-lock no-error. If not avail UBHeader then RUN Log in gh_LogActivity (9, ip-labelName). Else 2000-UBHeader: DO: … End. (no assignment to v-userbar when not avail)

# Impacted Systems

lib/userbar.p, printlabel.p (token replacement), UBHeader table.

# Traceability

- [[../Data Models/UserBar/UserBar - legacy]] — §5 userbar.p logic
- [[BR-UserBar-001-UBName-single-letter-A-Z-unique|BR-UserBar-001]]

# Assertions

- If UBHeader is not found for the given letter, userbar.p logs (Log 9, ip-labelName) and returns a blank string.
- No user-facing error is shown; the USERBARA–Z token is replaced with blank.

# Related Information

