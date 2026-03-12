---
Risk-Level: P0
Business-Rule-Id: BR-Goals-007
Deprecated: false
---

# Description

The Goal Update message (msg-goal-update.p) updates existing Goals from JSON (tt-GoalsScale: GoalID, GoalStatus, OrdNum, OrdRefLine, ProdCode, ScalesInGoal, CaseCount, NetWgt, LabelWgt). For each tt-GoalsScale row, the procedure finds Goals by GoalID; if available it assigns AllScaleTotalCount = tt-GoalsScale.CaseCount, AllScaleTotalNetWgt = tt-GoalsScale.NetWgt, AllScaleTotalLabelWgt = tt-GoalsScale.LabelWgt, GoalStatus = tt-GoalsScale.GoalStatus, and DateStatusChange = TODAY. If the GoalStatus changed from the original value, the goal ID and new status are appended to the output message string (GoalID|GoalStatus, comma-separated). Goals that do not exist are skipped (no create). op-MsgString is returned trimmed of trailing commas.

# Source

- progress-SLC/host/msg-goal-update.p: READ-JSON to tt-GoalsScale; FOR EACH tt-GoalsScale: FIND Goals by GoalID exclusive; IF AVAILABLE assign AllScaleTotalCount/CaseCount, AllScaleTotalNetWgt/NetWgt, AllScaleTotalLabelWgt/LabelWgt, GoalStatus, DateStatusChange; if v-OrigStatus <> new GoalStatus then op-MsgString = op-MsgString + GoalID + "|" + GoalStatus + ","; TRIM(op-MsgString,",")

# Impacted Systems

Goals table, host/msg-goal-update.p, host messaging.

# Traceability

- Update applies only to existing Goals rows; no new Goals are created by this message.

# Assertions

- AllScaleTotals and GoalStatus are overwritten from the message for the matching GoalID.
- Return string lists only goals whose status actually changed.

# Related Information

- [[BR-Goals-001-Status-values-and-transitions]]
- [[BR-Goals-002-LimitBy-and-totals]]
