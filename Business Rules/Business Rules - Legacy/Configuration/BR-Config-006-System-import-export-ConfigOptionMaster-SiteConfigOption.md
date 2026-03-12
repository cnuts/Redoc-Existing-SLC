---
Risk-Level: P0
Business-Rule-Id: BR-Config-006
Deprecated: false
---

# Description

System import and export (system/import.w, system/export.w) include ConfigOptionMaster and SiteConfigOption. ConfigOptionMaster: export file "configol.d"; import key ConfigOptionMaster.OptionName = TConfigOptionMaster.OptionName. Before import, PlantID OptionValue and AcceptableValues are read from ConfigOptionMaster and stored; after import, if no ConfigOptionMaster row exists for PlantID, one is created with those stored values. SiteConfigOption: export file "siteconf.d"; import key SiteConfigOption.OptionName = TSiteConfigOption.OptionName. Before SiteConfigOption import, values for Uniquenum, ScaleID, and PlantID are read from existing SiteConfigOption rows into variables; after import, ipSiteCfgRestore runs: it restores Uniquenum and ScaleID to SiteConfigOption (find or create by OptionName, set OptionValue). For PlantID it finds or creates SiteConfigOption; if the current OptionValue is not a valid integer or is 0, it shows "No valid option value detected for PlantID. Restoring previous value." and sets OptionValue to the saved v-StCfg-PlantID. Thus Site Config import preserves and restores Uniquenum, ScaleID, and PlantID with validation for PlantID.

# Source

- progress-SLC/system/import.w: ConfigOptionMaster — for each where OptionName = "PlantID" save v-Cfg-PlantID, v-Cfg-PlantID-Vals; import configol.d; if no ConfigOptionMaster for PlantID create with saved values. SiteConfigOption — for each save Uniquenum, ScaleID, v-StCfg-PlantID from OptionName; import siteconf.d; run ipSiteCfgRestore. ipSiteCfgRestore: restore Uniquenum, ScaleID; for PlantID find/create, if INT(OptionValue) error or 0 then message and OptionValue = v-StCfg-PlantID.
- progress-SLC/system/export.w: configol.d for ConfigOptionMaster; siteconf.d for SiteConfigOption.

# Impacted Systems

ConfigOptionMaster, SiteConfigOption, system/import.w, system/export.w, configol.d, siteconf.d.

# Traceability

- PlantID is given special handling so a zero or invalid value after import is replaced by the pre-import value.

# Assertions

- After Site Config import, Uniquenum, ScaleID, and PlantID are restored; PlantID is validated and restored if invalid or zero.

# Related Information

- [[BR-Config-001-OptionName-unique-and-SiteConfig]]
