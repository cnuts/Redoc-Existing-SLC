---
Risk-Level: P0
Business-Rule-Id: BR-Goals-011
Deprecated: false
---

# Description

Goals.GoalStatus values used on the CTS (SLC) are defined as global preprocessor constants in lib/prepr.i. GoalReady is ' ' (space); GoalInProcess is 'InProcess'; GoalComplete is 'Complete'; GoalAchieved is 'Achieved'; GoalCanceledAtCTS is 'Canceled'; GoalCanceledAtHost is 'Cancel'. These constants are used in host and print logic for status transitions, SentToServer handling, and exclusion from selection or purge (e.g. CanceledAtCTS goals skipped in SetCount-Complete and in purge/sync behavior). Ready denotes a goal not yet started; InProcess means in progress; Complete and Achieved both denote finished (Achieved used when acknowledging a completed goal to keep producing); Canceled at CTS vs Host distinguishes where the cancel originated.

# Source

- progress-SLC/lib/prepr.i: &GLOBAL-DEFINE GoalReady ' ', GoalInProcess 'InProcess', GoalComplete 'Complete', GoalAchieved 'Achieved', GoalCanceledAtCTS 'Canceled', GoalCanceledAtHost 'Cancel'

# Impacted Systems

Goals.GoalStatus, prepr.i, goals-setstatus.p, goals-setcomplete.p, goals-setcount-complete.p, weigh.w UpdateGoalStatus, s-goalreset.w, clearlogs.p, selection UIs.

# Traceability

- 12/13/02 ajh: GoalInProcess; 12/16/02 GoalReady; 01/08/03 GoalReady set to ' ' (space).

# Assertions

- All status comparisons in Goals logic use these preprocessor symbols, not literal strings.
- GoalAchieved and GoalComplete are treated the same for sync to GoalsHeader/GoalsDetail (both set header/detail to Complete).

# Related Information

- [[BR-Goals-001-Status-values-and-transitions]]
