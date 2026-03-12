---
Risk-Level: P0
Business-Rule-Id: BR-Config-005
Deprecated: false
---

# Description

The host update process (update-slc.w) imports configuration tables from files in the update directory. When the file is "configol.d", for each record it creates tt-ConfigOptionMaster, IMPORT tt-ConfigOptionMaster, finds ConfigOptionMaster by OptionName = tt-ConfigOptionMaster.OptionName under exclusive-lock; if not available creates ConfigOptionMaster; buffer-copies tt-ConfigOptionMaster to the database row and releases. When the file is "siteconf.d", for each record it creates tt-SiteConfigOption, IMPORT tt-SiteConfigOption, finds SiteConfigOption by OptionName = tt-SiteConfigOption.OptionName under exclusive-lock; if not available creates SiteConfigOption; buffer-copies tt-SiteConfigOption to the database row and releases. Import key for both is OptionName; existing rows are overwritten by the buffer-copy.

# Source

- progress-SLC/host/update-slc.w: WHEN "configol.d": CREATE tt-ConfigOptionMaster, IMPORT; FIND ConfigOptionMaster WHERE OptionName = tt-ConfigOptionMaster.OptionName exclusive no-error; IF NOT AVAILABLE CREATE; BUFFER-COPY tt-ConfigOptionMaster TO ConfigOptionMaster; RELEASE. WHEN "siteconf.d": CREATE tt-SiteConfigOption, IMPORT; FIND SiteConfigOption WHERE OptionName = tt-SiteConfigOption.OptionName exclusive no-error; IF NOT AVAILABLE CREATE; BUFFER-COPY tt-SiteConfigOption TO SiteConfigOption; RELEASE.

# Impacted Systems

ConfigOptionMaster, SiteConfigOption, host/update-slc.w, configol.d, siteconf.d.

# Traceability

- Same pattern as other update-slc table imports: find by key, create if missing, buffer-copy from temp-table.

# Assertions

- configol.d and siteconf.d are processed per record; each record is matched by OptionName and create-or-update is applied.

# Related Information

- [[BR-Config-001-OptionName-unique-and-SiteConfig]]
