---
Risk-Level: P0
Business-Rule-Id: BR-Config-004
Deprecated: false
---

# Description

At startup, services.p defines the temp-table TT-Site-Value (OptionName, OptionValue, primary index OptionName) and runs Load-TT-Site-Values. Load-TT-Site-Values iterates over SiteConfigOption (code may reference it as SiteConfig in the primary DB) and for each row creates a TT-Site-Value row with the same OptionName and OptionValue; it also adds TT-Site-Value rows for SecurityOverride and Palletize from the registry (Chickway section). f-Get-Site-Value-TT(OptionName, ConvertFunction, DefaultValue, OptionDesc, CreateSiteConfig): looks up TT-Site-Value and SiteConfigOption by OptionName; if SiteConfigOption is not available and CreateSiteConfig is true, runs Create-SiteConfig; if TT-Site-Value is not available it creates a TT-Site-Value row with OptionName and OptionValue = DefaultValue; returns TT-Site-Value.OptionValue. f-Set-Site-Value-TT(OptionName, OptionValue, ..., UpdateSiteConfig): updates or creates TT-Site-Value for OptionName with OptionValue; if UpdateSiteConfig is true, finds or creates SiteConfigOption and sets its OptionValue (creating via Create-SiteConfig if missing). Thus the TT is the in-session cache; reads can default and optionally create DB rows; writes can update both TT and SiteConfigOption when UpdateSiteConfig is true.

# Source

- progress-SLC/lib/services.p: DEFINE TEMP-TABLE TT-Site-Value; RUN Load-TT-Site-Values. Load-TT-Site-Values: FOR EACH SiteConfig, CREATE TT-Site-Value, ASSIGN from SiteConfig; GET-KEY-VALUE for SecurityOverride and Palletize, CREATE TT-Site-Value. f-Get-Site-Value-TT: FIND TT-Site-Value, FIND SiteConfig; if not avail SiteConfig and ip-CreateSiteConfig RUN Create-SiteConfig; if not avail TT-Site-Value CREATE TT-Site-Value with DefaultValue; RETURN TT-Site-Value.OptionValue. f-Set-Site-Value-TT: FIND/update or CREATE TT-Site-Value; if ip-UpdateSiteConfig FIND/Create SiteConfig, set OptionValue.

# Impacted Systems

TT-Site-Value, SiteConfigOption, ConfigOptionMaster (via Create-SiteConfig), lib/services.p, all callers of f-Get-Site-Value-TT and f-Set-Site-Value-TT.

# Traceability

- f-Rebuild-Site-Value-TT-From-Site repopulates TT-Site-Value from SiteConfigOption (no registry), used after globalassigns.i so cache matches DB.

# Assertions

- TT-Site-Value is populated from SiteConfigOption at load and can be repopulated via f-Rebuild-Site-Value-TT-From-Site.
- When UpdateSiteConfig is true, f-Set-Site-Value-TT keeps SiteConfigOption and TT-Site-Value in sync for that OptionName.

# Related Information

- [[BR-Config-001-OptionName-unique-and-SiteConfig]]
