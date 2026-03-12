---
Risk-Level: P0
Business-Rule-Id: BR-Operator-006
Deprecated: false
---

# Description

Host messages can create or update Operator and OperatorAppFunction from the production/host system. (1) host/msg-operator.p: The message body (ScaleMsg.MsgString) is written to temp\operator.d and imported into temp-table tt-Operator (LIKE Operator). For each tt-Operator, the procedure finds Operator by Operator.Operator (exclusive-lock); if not available it creates a new Operator and buffer-copies tt-Operator to Operator; else it buffer-copies tt-Operator to Operator (update). (2) host/msg-operator-security.p: The message body is written to temp\OperAppF.d and imported into tt-OperatorSecurity (LIKE OperatorAppFunction). For each row, it finds OperatorAppFunction by Operator and ID (exclusive-lock); if not available it creates a new OperatorAppFunction and buffer-copies; else buffer-copies (update). So the host can push Operator master data and OperatorAppFunction permission data to the SLC.

# Source

- progress-SLC/host/msg-operator.p — output to temp\operator.d put MsgString; input from temp\operator.d; repeat trans: create tt-Operator, import. do transaction: FIND Operator by tt-Operator.Operator; if not avail CREATE Operator and buffer-copy tt-Operator to Operator; else buffer-copy tt-Operator to Operator.
- progress-SLC/host/msg-operator-security.p — output to temp\OperAppF.d; input import tt-OperatorSecurity. do transaction: FIND OperatorAppFunction by Operator and ID; if not avail CREATE and buffer-copy; else buffer-copy.

# Impacted Systems

host/msg-operator.p, host/msg-operator-security.p, Operator table, OperatorAppFunction table, ScaleMsg, messaging host.

# Traceability

- [[BR-Operator-002-Operator-unique-AppFunction-mapping]] — Operator and OperatorAppFunction structure
- Host log records "Operator Updated" / "Operator Security Updated"

# Assertions

- msg-operator.p creates or updates Operator from the message payload (temp\operator.d).
- msg-operator-security.p creates or updates OperatorAppFunction from the message payload (temp\OperAppF.d).

# Related Information

