---
Risk-Level: P0
Business-Rule-Id: BR-Operator-002
Deprecated: false
---

# Description

Operator table primary key is Operator (login name), unique. OperatorAppFunction has composite primary key (Operator, ID) and unique index (ID, Operator); each Operator–AppFunction pair has at most one row with Permitted yes/no. AppFunction.ID is period-separated (e.g. Print.LabelPrint, Setup.Goals).

# Source

- [[../Data Models/Operator/Operator - legacy]] — Table structure, AppFunction examples
- [[../Data Models/0.Legacy Database Schema]] — Operator index Oper; OperatorAppFunction indexes OperatorID, ID-Operator; AppFunction ID-index

# Impacted Systems

Operator, OperatorAppFunction, AppFunction tables; login and permission loading.

# Traceability

- Operator CHAR(10) PK; OperatorAppFunction (Operator, ID) PK; AppFunction ID CHAR(50) PK

# Assertions

- Operator (login name) unique per record.
- (Operator, ID) unique in OperatorAppFunction.
- AppFunction defines all function IDs; OperatorAppFunction stores which operator has which function permitted.

# Related Information

