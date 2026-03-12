---
Risk-Level: P0
Business-Rule-Id: BR-Message-001
Deprecated: false
---

# Description

Msg table (labeled "Message") stores localized text by LangCode and MsgId. Primary key is (LangCode, MsgId) unique (index Msg). LangCode is character x(10) (e.g. "English", "Spanish"); MsgId is character x(30), max 60 (e.g. "PrintLabels.Quit", "Login.GlobalKillDateInEffect"). Msg holds the actual text (x(256), max 512); MsgLen is the display length limit. Used for UI labels, titles, and user-facing messages.

# Source

- [[../Data Models/Message/Message - legacy]] — §2.1 Schema, §2.3 Usage
- [[../Data Models/0.Legacy Database Schema]] — Msg table (LangCode, MsgId, Msg, MsgLen), index Msg yes yes LangCode, MsgId
- slc.df msg

# Impacted Systems

Msg table; get-Lang-Lbl, get-Lang-Msg (lib/get-lang-msg.i); UI labels, buttons, window titles, confirmations, errors; v-translate.w, import-msg-csv.w.

# Traceability

- MsgId convention: ProgramLogicalName.ChildObject (e.g. PrintLabels.Quit); get-Lang-Lbl returns SUBSTRING(Msg.Msg, 1, Msg.MsgLen)

# Assertions

- (LangCode, MsgId) is unique (primary key).
- LangCode and MsgId are required for lookup; Msg and MsgLen hold the text and display length.
- gLangCode (session/site) is typically passed as ip-LangCode so the UI uses the current language.

# Related Information

