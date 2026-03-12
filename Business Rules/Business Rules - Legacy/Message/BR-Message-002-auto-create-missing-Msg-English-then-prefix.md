---
Risk-Level: P0
Business-Rule-Id: BR-Message-002
Deprecated: false
---

# Description

When get-Lang-Lbl(ip-LangCode, ip-MsgId) does not find a Msg row for the requested LangCode and MsgId, it auto-creates one. In a transaction: look up English for ip-MsgId; if no English row exists, CREATE Msg with LangCode='English', MsgId=ip-MsgId, Msg=ip-MsgId, MsgLen=LENGTH(ip-MsgId). If ip-LangCode <> 'English', CREATE Msg for that language with Msg = first 2 chars of LangCode + ip-MsgId (to flag untranslated). MsgId is thus created on first use; English defaults to the key; other languages get a prefixed key until translated. get-Lang-Msg returns ip-MsgId if not available (no auto-create for get-Lang-Msg).

# Source

- [[../Data Models/Message/Message - legacy]] — §2.2 get-Lang-Lbl and get-Lang-Msg
- lib/get-lang-msg.i

# Impacted Systems

Msg table, get-Lang-Lbl, first-use message creation, v-translate.w (editing), add-to-runtime-db.p (2000-Msg seeding).

# Traceability

- get-Lang-Msg: same lookup by LangCode and MsgId; replaces [%1], [%2]... in Msg.Msg with ip-values (pipe-separated); if not available returns ip-MsgId

# Assertions

- Missing Msg for get-Lang-Lbl: create English row with Msg=MsgId if no English row; create ip-LangCode row with Msg = first 2 chars of LangCode + ip-MsgId when ip-LangCode <> 'English'.
- MsgId created on first use; other languages get prefixed key until translated.
- get-Lang-Msg does not auto-create; returns ip-MsgId when not found.

# Related Information

