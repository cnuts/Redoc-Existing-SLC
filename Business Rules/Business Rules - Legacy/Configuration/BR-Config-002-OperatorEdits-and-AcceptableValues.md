---
Risk-Level: P0
Business-Rule-Id: BR-Config-002
Deprecated: false
---

# Description

ConfigOptionMaster.OperatorEdits indicates whether operators are allowed to edit this option (vs. internal/admin only). AcceptableValues is a pipe-separated list of valid values (e.g. YES|NO, 02:00|22:00|24:00) used by UI for dropdowns and validation.

# Source

- [[../Data Models/Site Configuration Option/Site Configuration Options - legacy]] — §2.1 ConfigOptionMaster
- [[../Data Models/0.Legacy Database Schema]] — ConfigOptionMaster (OperatorEdits, AcceptableValues)

# Impacted Systems

ConfigOptionMaster, UI for site config and options that use AcceptableValues (e.g. lot validation in Modify).

# Traceability

- OperatorEdits logical; AcceptableValues character X(60)

# Assertions

- OperatorEdits controls whether operator can change the option value.
- AcceptableValues used for validation and UI dropdowns when present.

# Related Information

