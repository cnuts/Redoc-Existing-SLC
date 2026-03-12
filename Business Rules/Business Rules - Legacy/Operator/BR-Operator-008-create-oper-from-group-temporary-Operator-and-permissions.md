---
Risk-Level: P0
Business-Rule-Id: BR-Operator-008
Deprecated: false
---

# Description

When creating a temporary operator from a group (lib/create-oper-from-group.p), given ipGroup and ipUser: the procedure finds the Operator where Operator = ipGroup. If that group Operator does not exist, it returns. Otherwise it creates a new Operator row by buffer-copying the group Operator except the Operator field, and assigns Operator = ipUser. It then deletes all existing OperatorAppFunction rows for ipUser, and for each OperatorAppFunction where Operator = ipGroup it creates a new OperatorAppFunction row for ipUser (buffer-copy except Operator, assign Operator = ipUser). Finally it finds or creates ConfigOptionMaster with OptionName = "TemporaryADUsers" and appends ipUser to AcceptableValues (using CHR(2) as separator if values already exist). So AD/group-authenticated users get a temporary Operator record and a copy of the group's permissions; the user is tracked in TemporaryADUsers for cleanup on logout.

# Source

- progress-SLC/lib/create-oper-from-group.p — FIND Operator where Operator = ipGroup. If not avail return. CREATE buff-Operator, buffer-copy group Operator except Operator, assign Operator = ipUser. FOR EACH OperatorAppFunction where Operator = ipUser: DELETE. FOR EACH OperatorAppFunction where Operator = ipGroup: CREATE buff-OperAppFunc, buffer-copy except Operator, assign Operator = ipUser. FIND/CREATE ConfigOptionMaster OptionName "TemporaryADUsers", assign AcceptableValues = existing + CHR(2) + ipUser.

# Impacted Systems

Operator table, OperatorAppFunction table, ConfigOptionMaster (TemporaryADUsers), lib/create-oper-from-group.p, AD/group login flow.

# Traceability

- [[BR-Operator-002-Operator-unique-AppFunction-mapping]] — Operator and OperatorAppFunction
- [[BR-Operator-005-f-get-permission-CFSUSER-and-SecurityOverride-bypass]] — temporary operator gets permissions from group
- Comments in s-login: group permissions override local; temporary user removed on logout

# Assertions

- Group Operator must exist for create-oper-from-group to run; otherwise procedure returns.
- Temporary Operator row is created with Operator = ipUser; all OperatorAppFunction rows for ipUser are replaced by copies of the group's OperatorAppFunction rows.
- ipUser is added to ConfigOptionMaster "TemporaryADUsers" AcceptableValues for tracking.

# Related Information

