---
Risk-Level: P0
Business-Rule-Id: BR-Goals-012
Deprecated: false
---

# Description

The Acknowledge Goal screen (s-goalreset.w) allows the operator to acknowledge the current goal (gGoalsRowid). Supervisor permission is required (ValidateSecurity); if validation fails, the dialog does not complete. The Goals row is found by gGoalsRowid exclusive-lock; if not available, an error message "Missing goals record: system error: choose QUIT" is shown. If Goals.GoalStatus is Complete, the goal is set to Achieved: GoalStatus = GoalAchieved, DateStatusChange = today, SentToServer = no (so host/updategoals.p can send to production). Otherwise the goal is canceled at CTS: GoalStatus = GoalCanceledAtCTS, DateStatusChange = today, SentToServer = no. After update, gGoalsRowid is set to ? and a success message is shown (Goals.Achieved or Goals.Canceled per get-Lang-Lbl). local-exit is run to close the flow. Thus "Acknowledge" on a completed goal marks it Achieved and keeps producing; on a non-complete goal it cancels the goal.

# Source

- progress-SLC/goals/s-goalreset.w: ON CHOOSE btnOK: run ValidateSecurity; FIND Goals by gGoalsRowid exclusive; if not avail message "Missing goals record..."; if GoalStatus = Complete then assign GoalStatus=Achieved, DateStatusChange=today, SentToServer=no, vMsg=Goals.Achieved; else assign GoalStatus=CanceledAtCTS, DateStatusChange=today, SentToServer=no, vMsg=Goals.Canceled; gGoalsRowid=?; d-tsmsgbox success; run local-exit

# Impacted Systems

Goals table, s-goalreset.w, gGoalsRowid, ValidateSecurity (supervisor), host updategoals.p, get-Lang-Lbl (Goals.Reset.*, Goals.Achieved, Goals.Canceled).

# Traceability

- Achieved enables "keep producing" after goal complete; CanceledAtCTS stops the goal and triggers host sync.

# Assertions

- Only one Goals row is updated (that identified by gGoalsRowid).
- SentToServer = no after change so the host can sync the new status.
- Supervisor permission is required to acknowledge/reset the goal.

# Related Information

- [[BR-Goals-001-Status-values-and-transitions]]
- [[BR-Goals-011-Goal-status-constants-prepr]]
