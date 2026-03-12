---
Risk-Level: P0
Business-Rule-Id: BR-Goals-002
Deprecated: false
---

# Description

Goals are measured by LimitBy: Count, NetWgt, or LabelWgt. TargetLevel and optional Tolerance define the target; goal can be considered achieved at TargetLevel + Tolerance. CurrentCount, CurrentLabelWgt, CurrentNetWgt (this scale) and AllScaleTotalCount, AllScaleTotalNetWgt, AllScaleTotalLabelWgt are updated when serials are added or deleted via UpdateTotals/updatetotals2.

# Source

- [[../Data Models/Goal/Goal - Legacy]] — §2 (Goals table), §9 GetGoalProdTotal, §10 Updating Goals
- [[../Data Models/0.Legacy Database Schema]] — Goals (LimitBy, TargetLevel, Tolerance, Current*, AllScaleTotal*)

# Impacted Systems

Goals table, Serial/ItemSerial (GoalID), lib/updatetotals.p, lib/updatetotals2.p.

# Traceability

- PkgDesc3Size holds PrintedCount, PrintedWgt, Printed1, Printed2; SLC maintains these

# Assertions

- LimitBy in ('Count', 'NetWgt', 'LabelWgt').
- Goal status Complete when corresponding total >= TargetLevel (Count, NetWgt, or LabelWgt).
- Totals incremented/decremented by vAddDeleteFactor when serials added/deleted.
- opPdnTargetPlusTolerance = Goals.TargetLevel + Goals.Tolerance.

# Related Information

