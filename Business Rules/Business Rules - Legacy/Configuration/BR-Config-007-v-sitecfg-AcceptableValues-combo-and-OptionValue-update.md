---
Risk-Level: P0
Business-Rule-Id: BR-Config-007
Deprecated: false
---

# Description

In the site config viewer (viewers/v-sitecfg.w), the combo box cbAcceptableValues displays and updates the current option value. Its list-items are set from ConfigOptionMaster.AcceptableValues with delimiter "|". If the current SiteConfigOption.OptionValue is not in the AcceptableValues list (LOOKUP returns 0), the viewer appends that OptionValue to ConfigOptionMaster.AcceptableValues (with a leading "|" if AcceptableValues is non-empty) so the combo can show the current value; cbAcceptableValues:screen-value is set to SiteConfigOption.OptionValue. On value-changed, when vUpdatesPermitted is true and the new screen value differs from SiteConfigOption.OptionValue, the viewer logs the change (lib/log-changes.p), assigns SiteConfigOption.OptionValue = cbAcceptableValues:screen-value, and runs new-state ('NewSiteCfg'). When operator edits are not permitted, cbAcceptableValues is hidden and its label cleared. Thus the combo reflects AcceptableValues; the current value is always present in the list (added to AcceptableValues if missing); edits update SiteConfigOption.OptionValue and trigger state/link updates.

# Source

- progress-SLC/viewers/v-sitecfg.w: cbAcceptableValues DELIMITER "|". display-assign: if LOOKUP(SiteConfigOption.OptionValue, ConfigOptionMaster.AcceptableValues, '|') = 0 then ConfigOptionMaster.AcceptableValues = ConfigOptionMaster.AcceptableValues + (if > '' then '|' else '') + SiteConfigOption.OptionValue; cbAcceptableValues:list-items = ConfigOptionMaster.AcceptableValues; cbAcceptableValues:screen-value = SiteConfigOption.OptionValue. ON VALUE-CHANGED: if vUpdatesPermitted and SiteConfigOption.OptionValue <> cbAcceptableValues:screen-value then run log-changes, SLC.SiteConfigOption.OptionValue = cbAcceptableValues:screen-value, run new-state ('NewSiteCfg'). When not permitted, cbAcceptableValues:hidden = true, LABEL = "".

# Impacted Systems

viewers/v-sitecfg.w, ConfigOptionMaster.AcceptableValues, SiteConfigOption.OptionValue, lib/log-changes.p.

# Traceability

- AcceptableValues is pipe-delimited; the UI ensures the current value appears in the combo even if it was not originally in AcceptableValues.

# Assertions

- Saving a value from the combo updates SiteConfigOption.OptionValue only; ConfigOptionMaster.AcceptableValues may be updated for display when the current value is not in the list.

# Related Information

- [[BR-Config-002-OperatorEdits-and-AcceptableValues]]
