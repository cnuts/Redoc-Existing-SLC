---
Risk-Level: P0
Business-Rule-Id: BR-Config-008
Deprecated: false
---

# Description

The Find Site Config Option dialog (dialogs/d-sitecfgfind.w) and Find Master Config dialog (dialogs/d-mastercfgfind.w) perform a GTE lookup by OptionName and set a global rowid for the browser to reposition. d-sitecfgfind: user enters fiOptionName; on OK, FIND FIRST SiteConfigOption WHERE OptionName >= fiOptionName no-lock no-error. If available and OptionName <> fiOptionName, show message "No Config Option \"<fiOptionName>\", finding next". If not available, FIND LAST SiteConfigOption no-lock no-error; if available show "No site config Option with that name, finding previous". If available (from either find), gSiteCfgRowid = rowid(SiteConfigOption). If no SiteConfigOption exists at all, show "No site configs, Emergency, contact CFS". d-mastercfgfind: FIND FIRST ConfigOptionMaster WHERE OptionName >= fiOptionName no-lock no-error. If available and OptionName <> fiOptionName, show "No Config Option \"<fiOptionName>\", finding next"; gMasterCfgRowid = rowid(ConfigOptionMaster). If not available, show "No Master Configs, Emergency, contact CFS". poCancel is set false when a find is performed.

# Source

- progress-SLC/dialogs/d-sitecfgfind.w: FIND first SiteConfigOption where OptionName >= fiOptionName; if avail and <> fiOptionName d-tsmsgbox "finding next"; else if not avail find last SiteConfigOption, if avail d-tsmsgbox "finding previous"; if avail gSiteCfgRowid = rowid(SiteConfigOption); else d-tsmsgbox "No site configs, Emergency, contact CFS".
- progress-SLC/dialogs/d-mastercfgfind.w: FIND first ConfigOptionMaster where OptionName >= fiOptionName; if avail and <> fiOptionName d-tsmsgbox "finding next"; if avail gMasterCfgRowid = rowid(ConfigOptionMaster); else d-tsmsgbox "No Master Configs, Emergency, contact CFS".

# Impacted Systems

SiteConfigOption, ConfigOptionMaster, dialogs/d-sitecfgfind.w, dialogs/d-mastercfgfind.w, gSiteCfgRowid, gMasterCfgRowid, browsers that use these rowids.

# Traceability

- Same GTE pattern as other find dialogs; Site Config find has fallback to last record and distinct messages for "finding next", "finding previous", and no configs at all.

# Assertions

- gSiteCfgRowid and gMasterCfgRowid are set only when a row is found; the browser repositions based on the new state.

# Related Information

- [[BR-Config-001-OptionName-unique-and-SiteConfig]]
