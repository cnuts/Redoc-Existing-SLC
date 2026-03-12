---
Risk-Level: P0
Business-Rule-Id: BR-Message-005
Deprecated: false
---

# Description

set-tt-translations.p is invoked by host dbmaint.w to populate a shared temp-table (tt-TranslatedMsg) with translated message text without dbmaint itself depending on the database. Given an input message ID (ip-MsgID), the procedure creates one row in tt-TranslatedMsg with msg-id = ip-MsgID and translated-msg = get-lang-lbl(gLangCode, ip-MsgID). Thus the host can request a translation by message ID and receive the current language text via the shared tt, which dbmaint then uses (e.g. in ipTranslate).

# Source

- progress-SLC/host/set-tt-translations.p: DEF INPUT PARAM ip-MsgID AS c; DEFINE SHARED TEMP-TABLE tt-TranslatedMsg FIELD msg-id AS c FIELD translated-msg AS c INDEX i-msg-id IS PRIMARY msg-id; DO TRANSACTION: CREATE tt-TranslatedMsg, ASSIGN msg-id = ip-MsgID, translated-msg = get-lang-lbl(gLangCode, ip-MsgID)
- Invoked by host/dbmaint.w (which cannot contain DB-dependent code such as get-lang-lbl directly)

# Impacted Systems

tt-TranslatedMsg (shared), set-tt-translations.p, dbmaint.w, get-lang-lbl (lib/get-lang-msg.i), Msg table, gLangCode.

# Traceability

- Single row per call: one ip-MsgID produces one tt-TranslatedMsg row.
- translated-msg is the same value that get-Lang-Lbl would return for the current session language (including auto-create behavior; see [[BR-Message-002-auto-create-missing-Msg-English-then-prefix]]).

# Assertions

- tt-TranslatedMsg is populated by this procedure and consumed by the caller (dbmaint); the procedure does not open or query Msg itself beyond the get-lang-lbl include.
- Index i-msg-id on msg-id allows lookup by message ID in the host.

# Related Information

- [[BR-Message-001-LangCode-MsgId-unique]]
- [[BR-Message-002-auto-create-missing-Msg-English-then-prefix]]
