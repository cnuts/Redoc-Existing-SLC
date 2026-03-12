---
Risk-Level: P0
Business-Rule-Id: BR-Batch-005
Deprecated: false
---

# Description

Before purging BatchOrder rows, clearlogs.p ensures the site config "BatchOrder Clear Days" exists. It finds SiteConfig (primary DB) where OptionName = "BatchOrder Clear Days". If not available, it creates SLC.SiteConfig with OptionName = "BatchOrder Clear Days" and OptionValue = "7", then finds ConfigOptionMaster for that OptionName; if ConfigOptionMaster is not available it creates SLC.ConfigOptionMaster with OptionName = "BatchOrder Clear Days", OptionDescr = "BatchOrder Clear Days", OptionValue = "7", OperatorEdits = TRUE, and AcceptableValues = "1|2|3|4|5|6|7|8|9|10". If SiteConfig was found, v-BatchOrder-Clear-Days is set to INT(SiteConfig.OptionValue) (no-error). Thus the purge uses v-BatchOrder-Clear-Days (default 7 when config is created); the allowed range for operator edits is 1 through 10.

# Source

- progress-SLC/lib/clearlogs.p: DEF VAR v-BatchOrder-Clear-Days AS INT NO-UNDO INIT 7. FIND FIRST SiteConfig WHERE OptionName = "BatchOrder Clear Days" NO-ERROR. IF NOT AVAILABLE THEN CREATE SLC.SiteConfig, ASSIGN OptionName = "BatchOrder Clear Days", OptionValue = "7"; FIND ConfigOptionMaster; IF NOT AVAIL THEN CREATE ConfigOptionMaster, ASSIGN OptionName, OptionDescr, OptionValue = "7", OperatorEdits = TRUE, AcceptableValues = "1|2|3|4|5|6|7|8|9|10". ELSE ASSIGN v-BatchOrder-Clear-Days = INT(SLC.SiteConfig.OptionValue) NO-ERROR.

# Impacted Systems

SiteConfigOption (or SiteConfig), ConfigOptionMaster, lib/clearlogs.p, BatchOrder purge (BR-Batch-002).

# Traceability

- Default 7 and AcceptableValues 1–10 constrain how many days of BatchOrder history are retained when the option is operator-editable.

# Assertions

- v-BatchOrder-Clear-Days is used in the FOR EACH BatchOrder WHERE BatchDate < TODAY - v-BatchOrder-Clear-Days purge condition; if INT(OptionValue) fails, the initial value 7 remains.

# Related Information

- [[BR-Batch-002-BatchOrder-purge-by-clear-days]]
