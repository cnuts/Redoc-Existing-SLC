---
Risk-Level: P0
Business-Rule-Id: BR-Goals-008
Deprecated: false
---

# Description

goals-setstatus.p (SetStatus) processes Goals rows where GoalStatus is InProcess, Complete, CanceledAtCTS, or Achieved and SentToServer = no. For each such goal it syncs status to Production.GoalsHeader and Production.GoalsDetail, then sets Goals.SentToServer = yes. For InProcess: if GoalsHeader exists and its status is '' or Complete, set GoalsHeader and this scale's GoalsDetail (where ScaleID = gScaleID) to InProcess and DateStatusChange = today. For CanceledAtCTS: set this scale's GoalsDetail to CanceledAtCTS. For Complete or Achieved: set GoalsHeader to Complete and DateStatusChange = today; then set every GoalsDetail for that goal (status '' or InProcess) to Complete. After all header/detail updates, the Goals row is updated with SentToServer = yes. The SentToServer = no condition prevents re-sending after a goal is complete or canceled unless status changes again.

# Source

- progress-SLC/host/goals-setstatus.p: For each Goals where (GoalStatus in Complete/CanceledAtCTS/Achieved/InProcess) and SentToServer = no; case GoalStatus: when InProcess update GoalsHeader and GoalsDetail (this scale) to InProcess; when CanceledAtCTS update this scale GoalsDetail to CanceledAtCTS; when Complete/Achieved update GoalsHeader to Complete, all Details ("" or InProcess) to Complete; then find current b-Goals exclusive, assign SentToServer = yes

# Impacted Systems

Goals table, Production.GoalsHeader, Production.GoalsDetail, goals-setstatus.p, host/Progress-host.w.

# Traceability

- First scale to complete drives GoalsHeader and all details to Complete.
- 04/19/07: If last case for a goal is canceled, goal remains closed (no change in this procedure).

# Assertions

- Only Goals with SentToServer = no are processed; after processing, SentToServer = yes.
- GoalsDetail is updated only for the current scale (gScaleID) for InProcess and CanceledAtCTS; for Complete/Achieved all matching details are updated.

# Related Information

- [[BR-Goals-001-Status-values-and-transitions]]
