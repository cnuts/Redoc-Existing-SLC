---
Risk-Level: P0
Business-Rule-Id: BR-CrossRef-001
Deprecated: false
---

# Description

The CrossRef table has composite unique primary key (Application, ID). Each Application/ID pair is unique. The table stores flexible key-value data: Application (category, e.g. SellByOffset, ZuluDeptsShift, Counter-Goal), ID (identifier within category, e.g. product code, shift, GoalID), Descr (value or data).

# Source

- [[../Data Models/Cross References/Understanding Cross References]]
- [[../Data Models/0.Legacy Database Schema]] — CrossRef, index UniqueAppCommIdx (Application, ID) unique

# Impacted Systems

CrossRef table; Product (Family-*, Product-MFGID), Modify (ZuluDeptsShift), Goals (Counter-Goal), SellByOffset, ValidMachineIDs, and other Application usages.

# Traceability

- UniqueAppCommIdx yes yes Application ASCENDING, ID ASCENDING
- Application X(15), ID X(30), Descr X(30)

# Assertions

- (Application, ID) unique.
- Used for Counter-Goal (ID = STRING(Goals.GoalID)), ZuluDeptsShift (ID = gShift), Family-ProductCode, Product-MFGID, SellByOffset, and other key-value lookups.

# Related Information

