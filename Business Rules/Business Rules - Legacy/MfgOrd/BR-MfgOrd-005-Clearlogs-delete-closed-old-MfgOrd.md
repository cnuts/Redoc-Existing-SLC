---
Risk-Level: P0
Business-Rule-Id: BR-MfgOrd-005
Deprecated: false
---

# Description

The clearlogs process deletes MfgOrd rows that are not Open and that are either undated (MfgOrdDate = ?) or older than a configured number of days. The retention period is read from SiteConfigOption "Clear-MfgOrd-Days" (default 7); if the option is missing, it is created with value "7" and ConfigOptionMaster is created with OptionDescr "Delete MfgOrd if older than date and not Open". Deletion criteria: MfgOrd.MfgOrdStatus <> "Open" AND (MfgOrd.MfgOrdDate = ? OR MfgOrd.MfgOrdDate < TODAY - v-MfgOrd-Clear-Days).

# Source

- progress-SLC/lib/clearlogs.p: FIND SiteConfig OptionName = "Clear-MfgOrd-Days"; if avail use INT(OptionValue) else CREATE SiteConfig/ConfigOptionMaster with OptionValue "7", OptionDescr "Delete MfgOrd if older than date and not Open"; FOR EACH MfgOrd EXCLUSIVE-LOCK WHERE MfgOrdStatus <> "Open" AND (MfgOrdDate = ? OR MfgOrdDate < TODAY - v-MfgOrd-Clear-Days): DELETE MfgOrd

# Impacted Systems

MfgOrd table, SiteConfigOption (Clear-MfgOrd-Days), ConfigOptionMaster (Clear-MfgOrd-Days), clearlogs.p.

# Traceability

- Only closed (or non-Open) orders are eligible for deletion; Open orders are never deleted by this rule.
- Date filter: null date or date before (TODAY - Clear-MfgOrd-Days).

# Assertions

- v-MfgOrd-Clear-Days defaults to 7 and is overridden by SiteConfigOption.OptionValue when present.
- ConfigOptionMaster.OperatorEdits = yes for Clear-MfgOrd-Days; AcceptableValues = "7" when created.

# Related Information

- [[BR-MfgOrd-001-ProdCode-and-Serial-relationship]]
- [[BR-MfgOrd-002-Host-MfgOrd-message-create-update-close]]
- [[BR-Config-001-OptionName-unique-and-SiteConfig]]
