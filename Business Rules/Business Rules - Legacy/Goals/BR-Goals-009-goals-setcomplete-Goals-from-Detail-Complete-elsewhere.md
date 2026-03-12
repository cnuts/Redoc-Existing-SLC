---
Risk-Level: P0
Business-Rule-Id: BR-Goals-009
Deprecated: false
---

# Description

goals-setcomplete.p (SetComplete) updates the local Goals row when this scale's GoalsDetail indicates the goal was completed on another scale. For each Production.GoalsDetail where ScaleID = gScaleID and GoalStatus = Complete, the procedure finds the Goals row with the same GoalID and status '' (Ready) or InProcess. If found, it sets Goals.GoalStatus = Complete (from GoalsDetail.GoalStatus) and Goals.DateStatusChange = today. This allows the SLC Goals table to reflect completion that was first recorded in Production.GoalsDetail (e.g. by another scale). Goals that are already Complete or in other statuses are not modified by this path.

# Source

- progress-SLC/host/goals-setcomplete.p: For each GoalsDetail where ScaleID = gScaleID and GoalStatus = Complete; find Goals where GoalID = GoalsDetail.GoalID and (GoalStatus = '' or InProcess) exclusive; if avail assign Goals.GoalStatus = b-GoalsDetail.GoalStatus (Complete), Goals.DateStatusChange = today

# Impacted Systems

Goals table, Production.GoalsDetail, goals-setcomplete.p, host work cycle.

# Traceability

- "Completed elsewhere" flow: Production.GoalsDetail.GoalStatus = Complete on this scale implies the goal was completed (possibly on another scale); local Goals is updated to Complete so UI and totals stay in sync.

# Assertions

- Only Goals in Ready or InProcess are updated to Complete by this procedure.
- Only this scale's GoalsDetail rows (gScaleID) are read; completion is propagated from Production to SLC Goals.

# Related Information

- [[BR-Goals-001-Status-values-and-transitions]]
- [[BR-Goals-008-goals-setstatus-SentToServer-sync-GoalsHeader-GoalsDetail]]
