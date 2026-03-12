---
Risk-Level: P0
Business-Rule-Id: BR-Modify-003
Deprecated: false
---

# Description

The Modify screen (s-modify.w) enforces config/site limits (e.g. gMaxBackDate for pack date) and validates Lot against ConfigOptionMaster.AcceptableValues when lot is restricted. Fields can be disabled by permission: Print.Modify.ZuluCapture, Print.Modify.Price, Print.Modify.Lot, Print.Modify.KillDate, Print.Modify.PackDate, Print.Modify.VarOffset, Print.Modify.Tare.

# Source

- [[../Data Models/Modify/Modify - legacy]] — §9 Modify Screen (s-modify.w), §12 Business Rules Summary
- sobjects/s-modify.w; messages PrintLabels.Modify.Msg.ConfigMaxExceed, PrintLabels.Modify.MissingLots

# Impacted Systems

s-modify.w, OperatorAppFunction (Print.Modify.*), ConfigOptionMaster (AcceptableValues for lot), site config (MaximumBackDate).

# Traceability

- Permissions: Print.Modify.ZuluCapture, Print.Modify.Price, Print.Modify.Lot, Print.Modify.KillDate, Print.Modify.PackDate, Print.Modify.VarOffset, Print.Modify.Tare

# Assertions

- Modify screen respects security; only fields the operator is permitted to change are editable.
- Lot validated against AcceptableValues when lot is restricted.
- Pack date and other offsets limited by gMaxBackDate and config.

# Related Information

