---
Risk-Level: P0
Business-Rule-Id: BR-Message-007
Deprecated: false
---

# Description

The Translate viewer (v-translate.w) and browser (b-translate.w) treat English as the base language. The browser query lists Msg where LangCode = "English" (and KEY-PHRASE). In v-translate.w, when the user deletes a message (local-delete-record), the procedure first deletes all Msg rows where LangCode ne "English" and MsgId equals the current record's MsgId (removing all translations for that message), then dispatches the standard delete-record to remove the English row. Thus the English row is deleted last; all other language rows for that MsgId are removed first. Saving or creating in v-translate updates or creates Msg by LangCode and MsgId with Msg and MsgLen from the screen; new translations can be created for a selected language (CMB-Lang) and MsgId.

# Source

- progress-SLC/viewers/v-translate.w: local-delete-record — message "Delete will delete the message with its associated translations..."; FOR EACH b-Msg WHERE b-Msg.LangCode ne "English" AND b-Msg.MsgId = Msg.MsgId EXCLUSIVE-LOCK: DELETE b-Msg; then RUN dispatch delete-record. local-assign-record: FIND b-Msg by LangCode and MsgId; if avail update Msg and MsgLen else CREATE b-Msg with LangCode (v-newMsg or CMB-Lang), MsgId, Msg, MsgLen. local-display-fields: find b-Msg by CMB-Lang and Msg.MsgId, Alien-Msg = b-Msg.Msg
- progress-SLC/browsers/b-translate.w: OPEN QUERY br_table FOR EACH Msg WHERE Msg.LangCode = "English" SHARE-LOCK

# Impacted Systems

Msg table, v-translate.w, b-translate.w, translation UI (add/update/delete by language and MsgId).

# Traceability

- Delete order: non-English rows for the same MsgId first, then English row, so referential consistency and UI base list remain valid.
- Base list for translation is English-only; other languages are edited by selecting language (CMB-Lang) and MsgId.

# Assertions

- Only Msg rows with LangCode = "English" appear in the translate browser list.
- Deleting from the viewer removes every translation (every LangCode) for that MsgId, then the English record.
- New translation creation uses MsgId from the current English row and LangCode from CMB-Lang (or screen); Msg and MsgLen come from the alien/screen fields.

# Related Information

- [[BR-Message-001-LangCode-MsgId-unique]]
- [[BR-Message-002-auto-create-missing-Msg-English-then-prefix]]
