---
Risk-Level: P0
Business-Rule-Id: BR-Config-003
Deprecated: false
---

# Description

create-sitecfg.p creates configuration rows in a fixed order and reports failure via an output parameter. It is invoked with ip-OptionName, ip-OptionDesc, ip-OptionDefaultValue and returns op-SiteCfgCreatedFailure. If a SiteConfigOption row already exists for that OptionName, the procedure returns immediately (no changes, op-SiteCfgCreatedFailure remains no). Otherwise it creates SiteConfigOption with OptionName and OptionValue = ip-OptionDefaultValue. If that create raises error-status:error, it sets op-SiteCfgCreatedFailure = yes and returns. Then it finds ConfigOptionMaster by OptionName; if not available it creates ConfigOptionMaster with OptionName, OptionDescr = ip-OptionDesc, OptionValue = ip-OptionDefaultValue, AcceptableValues = ip-OptionDefaultValue, OperatorEdits = yes. If that create raises error-status:error it sets op-SiteCfgCreatedFailure = yes and returns. Thus SiteConfigOption is always created first when missing; ConfigOptionMaster is created only when missing; any create error is reported via op-SiteCfgCreatedFailure.

# Source

- progress-SLC/lib/create-sitecfg.p: FIND SiteConfigOption WHERE OptionName = ip-OptionName; if avail return. Else CREATE SiteConfigOption, ASSIGN OptionName, OptionValue = ip-OptionDefaultValue; on error op-SiteCfgCreatedFailure = yes, return. FIND ConfigOptionMaster WHERE OptionName = ip-OptionName; if not avail CREATE ConfigOptionMaster, ASSIGN OptionName, OptionDescr, OptionValue, AcceptableValues = ip-OptionDefaultValue, OperatorEdits = yes; on error op-SiteCfgCreatedFailure = yes, return.

# Impacted Systems

ConfigOptionMaster, SiteConfigOption, lib/create-sitecfg.p, callers (e.g. services.p Create-SiteConfig).

# Traceability

- Create order: SiteConfigOption then ConfigOptionMaster. Output op-SiteCfgCreatedFailure used by services.p to log to temp/site-cfg-err.log and display message.

# Assertions

- SiteConfigOption is never created if a row already exists for OptionName.
- ConfigOptionMaster is created only when no row exists for OptionName and only after SiteConfigOption has been created for this call.

# Related Information

- [[BR-Config-001-OptionName-unique-and-SiteConfig]]
