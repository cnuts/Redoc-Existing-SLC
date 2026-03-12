---
Risk-Level: P0
Business-Rule-Id: BR-Serial-005
Deprecated: false
---

# Description

If GoalID is not 0 (or not null), GoalID must exist in the Goals table. If PalletNum is specified, it must exist in PalletHdr. GoalID can be 0 for off-goal production.

# Source

- [[../Data Models/Serial/Serial - legacy]] — State Rules (Goal Link Optional), Data Integrity Rules, Data Validation Checklist
- [[../Data Models/Goal/Goal - Legacy]] — Serial.GoalID set from gGoalsRowid

# Impacted Systems

Serial table, Goals table, PalletHdr/PalletDtl, createserial.p, updatetotals/updatetotals2.

# Traceability

- [[../Data Models/0.Legacy Database Schema]] — Serial (GoalID, PalletNum)
- Validation checklist: If GoalID ≠ 0, GoalID exists in Goals; If PalletNum specified, exists in PalletHdr

# Assertions

- GoalID optional (0 or null = off-goal).
- When GoalID is set, it must reference an existing Goals.GoalID.
- When PalletNum is set, it must reference an existing PalletHdr.PalletNum.
- ItemCode, if specified, must exist in Item table.

# Related Information

