---
Risk-Level: P0
Business-Rule-Id: BR-Config-001
Deprecated: false
---

# Description

ConfigOptionMaster.OptionName is the primary key (unique). SiteConfigOption has one row per OptionName with OptionValue; OptionName is the primary key and must match an option defined (or definable) in ConfigOptionMaster. If code requests an option missing from SiteConfigOption, the system can auto-create a row (and optionally ConfigOptionMaster) via create-sitecfg.p using a caller-supplied default and description.

# Source

- [[../Data Models/Site Configuration Option/Site Configuration Options - legacy]] — §2 Schema, §3.5 Create-SiteConfig
- [[../Data Models/0.Legacy Database Schema]] — ConfigOptionMaster, SiteConfigOption

# Impacted Systems

ConfigOptionMaster, SiteConfigOption, TT-Site-Value, f-get-session-parm, f-Get-Site-Value-TT, f-set-session-parm, create-sitecfg.p, add-to-runtime-db.p.

# Traceability

- ConfigOptionMaster: OptionName PK, OptionDescr, OptionValue, AcceptableValues, OperatorEdits
- SiteConfigOption: OptionName PK, OptionValue

# Assertions

- OptionName unique in both tables.
- Writes (f-set-session-parm, f-Set-Site-Value-TT with UpdateSiteConfig) update both SiteConfigOption and TT-Site-Value when flag set.
- Create-SiteConfig: creates SiteConfigOption and optionally ConfigOptionMaster when option missing.

# Related Information

