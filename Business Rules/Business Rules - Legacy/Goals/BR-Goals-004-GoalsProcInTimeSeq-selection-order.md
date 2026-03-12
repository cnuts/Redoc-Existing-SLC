---
Risk-Level: P0
Business-Rule-Id: BR-Goals-004
Deprecated: false
---

# Description

When gGoalsProcInTimeSeq is true, the operator must select goals in target date/time order. CheckTargetTime finds the earliest goal by index TgtDateTimeGoal where status is Ready, In Process, or Complete; if the selected goal's rowid is not that earliest goal, the system shows a message that the user must select the earliest goal and sets poOK = no.

# Source

- [[../Data Models/Goal/Goal - Legacy]] — §12 CheckTargetTime and GoalsProcInTimeSeq
- Used in w-select-fresh-goals.w, w-select-batch-goals.w, w-select-goal-btns.w, select-order-detail.w, order-lines-last-used.w, b-open-goals.w

# Impacted Systems

Goal selection UI, session global gGoalsProcInTimeSeq, Goals index TgtDateTimeGoal (TgtCompDate, TgtCompTime, GoalID).

# Traceability

- If gGoalsProcInTimeSeq is false, poOK = yes and no check is done

# Assertions

- When GoalsProcInTimeSeq is enabled, selection is restricted to the earliest goal by TgtCompDate, TgtCompTime.
- CheckTargetTime returns poOK = no if user did not select that earliest goal.

# Related Information

