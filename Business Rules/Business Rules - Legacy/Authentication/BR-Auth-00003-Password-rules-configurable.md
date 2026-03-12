---
Risk-Level: P0
Business-Rule-Id: BR-Auth-00003
Deprecated: false
---

# Description

Password rules must be configurable:

- Specify a min/max length
- Acceptable symbols, numbers, etc.
- Weak password using whitelist/blacklist
- Last three passwords must not be reused.

# Source

- Security policy / redevelopment requirements
- [[../Business Rules/Authentication/BR-Auth-00003- Password rules must be configurable..]]

# Impacted Systems

Login service, operator credential management, password change flows.

# Traceability

# Assertions

- Minimum and maximum password length must be configurable.
- Allowed character set (symbols, numbers) must be configurable.
- Weak-password list (whitelist/blacklist) must be configurable.
- User must not reuse any of the last three passwords when changing password.

# Related Information

