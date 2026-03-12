---
Risk-Level: P0
Business-Rule-Id: BR-Goals-001
Deprecated: false
---

# Description

GoalStatus values are defined in lib/prepr.i: Ready = ' ' (space), InProcess = 'InProcess', Complete = 'Complete'. Other statuses in code include CANCELED, GoalCanceledAtCTS. When AllScaleTotalCount (or NetWgt/LabelWgt per LimitBy) reaches TargetLevel, GoalStatus is set to 'Complete' and SentToServer = no. When a goal was Complete and totals later fall at or below TargetLevel (e.g. serials deleted), GoalStatus is set to 'InProcess' and SentToServer = no.

# Source

- [[../Data Models/Goal/Goal - Legacy]] — §3 Goal Status Values, §10.2 Factor and Status Logic
- lib/prepr.i (GoalReady, GoalInProcess, GoalComplete)

# Impacted Systems

Goals table, updatetotals.p, updatetotals2.p, weigh.w (UpdateGoalStatus), goal selection UI.

# Traceability

- LimitBy = 'Count': AllScaleTotalCount >= TargetLevel → Complete; was Complete and AllScaleTotalCount <= TargetLevel → InProcess
- LimitBy = 'NetWgt' / 'LabelWgt': same logic with AllScaleTotalNetWgt / AllScaleTotalLabelWgt

# Assertions

- GoalStatus in (Ready ' ', InProcess, Complete, CANCELED, GoalCanceledAtCTS as used).
- Status transition to Complete when total (per LimitBy) >= TargetLevel.
- Status transition back to InProcess when total falls <= TargetLevel after being Complete.
- Goals with status CANCELED (or canceled at CTS) are excluded from host sync and from certain selection lists.

# Related Information

