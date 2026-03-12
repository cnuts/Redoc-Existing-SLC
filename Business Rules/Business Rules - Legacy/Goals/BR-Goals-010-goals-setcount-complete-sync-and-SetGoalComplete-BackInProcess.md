---
Risk-Level: P0
Business-Rule-Id: BR-Goals-010
Deprecated: false
---

# Description

goals-setcount-complete.p (SetCount-Complete) syncs current counts and weights between Goals, Production.GoalsDetail (this scale), and Production.GoalsHeader, then updates Goals.AllScaleTotals and status. Goals with GoalStatus = CanceledAtCTS are skipped. For each other Goals: GoalsDetail (this scale) is updated so CurrentCount, CurrentLabelWgt, CurrentNetWgt equal Goals; GoalsHeader is updated by adding the difference (CountDiff, LabelWgtDiff, NetWgtDiff) to its CurrentCount, CurrentLabelWgt, CurrentNetWgt. Then UpdateGoalAllScaleTotAndStatus runs: it copies AllScaleTotalCount, AllScaleTotalLabelWgt, AllScaleTotalNetWgt from Production.GoalsHeader to Goals. Based on LimitBy and TargetLevel, it calls SetGoalComplete (when total >= TargetLevel: GoalStatus = Complete, DateStatusChange = today, SentToServer = no) or SetGoalBackInProcess (when goal was Complete and total is now below TargetLevel: GoalStatus = InProcess, DateStatusChange = today, SentToServer = no). Re-opening to InProcess supports the case where a case was canceled and the goal falls back under target. A no-wait loop with a max wait (vMaxTimeToWaitMSGoals) is used when locking Goals for update.

# Source

- progress-SLC/host/goals-setcount-complete.p: For each Goals where GoalStatus <> CanceledAtCTS; sync GoalsDetail from Goals (CurrentCount/LabelWgt/NetWgt); add diff to GoalsHeader; UpdateGoalAllScaleTotAndStatus: copy AllScaleTotals from GoalsHeader to Goals; case LimitBy: Count/LabelWgt/NetWgt if total >= TargetLevel then SetGoalComplete else SetGoalBackInProcess. SetGoalComplete: GoalStatus=Complete, DateStatusChange=today, SentToServer=no. SetGoalBackInProcess: if GoalStatus=Complete then GoalStatus=InProcess, DateStatusChange=today, SentToServer=no

# Impacted Systems

Goals table, Production.GoalsHeader, Production.GoalsDetail, goals-setcount-complete.p, lib/updatetotals.p (AllScaleTotal updated there on print; this syncs from Header).

# Traceability

- 05/16/07: updatetotals.p now updates AllScaleTotal; SetCount-Complete still syncs from GoalsHeader so multi-scale totals are correct.
- 04/19/07: CanceledAtCTS goals left alone; if reactivated, later logic catches up.

# Assertions

- Goals with status CanceledAtCTS are not updated by SetCount-Complete.
- SetGoalBackInProcess runs only when current GoalStatus is Complete and total is below TargetLevel.
- SentToServer = no when status changes to Complete or back to InProcess so host can be notified.

# Related Information

- [[BR-Goals-001-Status-values-and-transitions]]
- [[BR-Goals-002-LimitBy-and-totals]]
