---
Risk-Level: P0
Business-Rule-Id: BR-Message-006
Deprecated: false
---

# Description

The Import Message CSV utility (lib/import-msg-csv.w) reads a CSV file with columns: message ID, English text, length, and translated message. The user specifies a destination file path and a target language (fiLangCode, e.g. French). For each CSV row: if no English Msg exists for that MsgId, one is created with MsgId, LangCode = 'English', Msg = MsgId (key as text), and MsgLen from the CSV or default. Then the procedure finds or creates the Msg row for the target language (fiLangCode) and sets Msg.Msg to the translated value from the CSV; MsgLen is taken from the existing English row when present, otherwise from the CSV. This supports bulk translation import by language.

# Source

- progress-SLC/lib/import-msg-csv.w: INPUT STREAM s-MsgCSV DELIMITER ',' vMsgID, vMsgEnglish, vMsgLen, vMsg; FIND b-Msg English by MsgID; IF NOT AVAIL create b-Msg (MsgID, LangCode='English', Msg=MsgID, MsgLen); FIND msg by MsgID and fiLangCode; IF NOT AVAIL CREATE msg (MsgID, LangCode=fiLangCode, MsgLen from b-Msg or vMsgLen); ASSIGN Msg.Msg = vMsg (translated); vCtrUpdated incremented. UI note: "CSV must have cols = msg ID,English,length, translated msg"

# Impacted Systems

Msg table, import-msg-csv.w, CSV file format (msg ID, English, length, translated msg), target language fiLangCode.

# Traceability

- English row is created only when missing; Msg text is set to MsgID as placeholder when creating English from CSV.
- Target language row is created if missing; in all cases Msg.Msg is updated to the CSV translated value for that row.

# Assertions

- CSV columns are: vMsgID (message ID), vMsgEnglish (unused in update logic but in file), vMsgLen (length), vMsg (translated message for target language).
- fiLangCode defines the language code for the created/updated translation rows (e.g. French).
- Repeated runs with the same CSV and language will update existing Msg.Msg for that language.

# Related Information

- [[BR-Message-001-LangCode-MsgId-unique]]
- [[BR-Message-002-auto-create-missing-Msg-English-then-prefix]]
