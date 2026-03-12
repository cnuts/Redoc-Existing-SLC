---
Risk-Level: P0
Business-Rule-Id: BR-Item-002
Deprecated: false
---

# Description

ItemSerial must be tied to a case: CaseSerialNum and ProductCode must match a Serial (case). ItemSerial.GoalID is set when gGoalsRowid is available (produce-to-goal). GetNextItemSerial generates unique item serial numbers (Plant+Scale+Julian+"00"+ItemSequence).

# Source

- [[../Data Models/Item/Item - legacy]] — §5 (CreateItemSerial, GetNextItemSerial), §8.2 Application rules
- [[../Data Models/Goal/Goal - Legacy]] — ItemSerial.GoalID from gGoalsRowid

# Impacted Systems

ItemSerial table, createitemserial.p, CancelItemSerials, Serial.ItemWgtList.

# Traceability

- createitemserial.p; GetNextItemSerial format; CancelItemSerials creates D rows mirroring A rows

# Assertions

- ItemSerial.CaseSerialNum and ProductCode must reference an existing Serial (case).
- When producing to goal, ItemSerial.GoalID set from current goal.
- CancelItemSerials: for a given case, create delete (D) ItemSerial rows that mirror add (A) rows (negated weights).

# Related Information

