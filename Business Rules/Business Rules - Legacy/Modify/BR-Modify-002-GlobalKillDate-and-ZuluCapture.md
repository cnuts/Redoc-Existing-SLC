---
Risk-Level: P0
Business-Rule-Id: BR-Modify-002
Deprecated: false
---

# Description

When site config GlobalKillDateInEffect is true, Modify.KillDateOffset is set from site GlobalKillDate so all products share one kill date. When ZuluCapture is true, ZuluDept in ModText1 is set (from global or shift config) and validated against CrossRef ZuluDeptsShift; invalid value is replaced with first valid and user is notified.

# Source

- [[../Data Models/Modify/Modify - legacy]] — §10 Create-Modify and Goals / Global Kill Date / Zulu
- weigh-create-modify.p: GlobalKillDateInEffect, ZuluCapture, ZuluFromGlobal, SetZuluDept, CrossRef 'ZuluDeptsShift'

# Impacted Systems

Modify, tt-Modify, SiteConfigOption (GlobalKillDateInEffect, GlobalKillDate, ZuluCapture, ZuluFromGlobal, ZuluGlobalDeptShift), CrossRef (ZuluDeptsShift + gShift).

# Traceability

- Modify.KillDateOffset = v-GlobalKillDate - TODAY when GlobalKillDateInEffect
- ZuluDept validated against CrossRef Application 'ZuluDeptsShift', ID = gShift (Descr = comma-separated valid depts)

# Assertions

- GlobalKillDateInEffect: when true, Modify.KillDateOffset set from site GlobalKillDate.
- ZuluCapture: when true, ZuluDept in ModText1 set and validated against CrossRef ZuluDeptsShift; invalid replaced with first valid, message shown.
- When producing to goal (gProduceToWO), UpdateFromGoal copies goal fields (CasesPerPallet, CustomerName, OrdNum, OrdRefLine, PackDateOffset, UnitPrice, VarOffset) into tt-Modify when goal supplies non-? values.

# Related Information

