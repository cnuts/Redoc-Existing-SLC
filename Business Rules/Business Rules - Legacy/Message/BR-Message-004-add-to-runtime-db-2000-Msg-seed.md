---
Risk-Level: P0
Business-Rule-Id: BR-Message-004
Deprecated: false
---

# Description

The 2000-Msg procedure in add-to-runtime-db.p seeds or updates the Msg table from internal arrays (v-MsgID, v-MsgLangCode, v-Msg, v-MsgLen). For each array index from 1 up to 100, processing stops when v-MsgID is blank. For each entry, Msg is found by MsgId and LangCode; if not available a new Msg row is created, otherwise the existing row is updated. All four fields (LangCode, MsgId, Msg, MsgLen) are set from the arrays. This provides initial or bulk message data for localization (e.g. PrintDate, PackDate, PrintLabels.*, Configs.*).

# Source

- progress-SLC/lib/add-to-runtime-db.p: PROCEDURE 2000-Msg; DO v-Entry = 1 TO 100; IF v-MsgID[v-Entry] = '' THEN LEAVE; FIND msg WHERE Msg.MsgId = v-MsgID[v-Entry] AND Msg.LangCode = v-MsgLangCode[v-Entry] EXCLUSIVE-LOCK NO-ERROR; IF NOT AVAIL msg THEN CREATE msg ASSIGN LangCode, Msg, MsgId, MsgLen from arrays; ELSE ASSIGN same; RELEASE msg; END

# Impacted Systems

Msg table, add-to-runtime-db.p (run at startup or maintenance), v-MsgID/v-MsgLangCode/v-Msg/v-MsgLen arrays.

# Traceability

- Key for lookup: (MsgId, LangCode) matching (v-MsgID[v-Entry], v-MsgLangCode[v-Entry]).
- MsgLen is stored as integer from v-MsgLen array (format "->,>>>,>>9" in schema).

# Assertions

- Maximum 100 Msg entries are processed per run of 2000-Msg; empty v-MsgID terminates the loop.
- CREATE assigns all four fields; UPDATE assigns the same four fields (full row refresh from arrays).

# Related Information

- [[BR-Message-001-LangCode-MsgId-unique]]
