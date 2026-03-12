# Message Concept and Usage in the legacy SLC Application

This document describes the **Message** concept in full: the **Msg** table (localization/translation), **CommHdr**/**CommDtl** (host and in-app messaging), **ErrorMsgs**, **Goals.AddMsg1/2/3**, **MailID**, and how messages relate to Operator, Goals, Label, Host, and other concepts.

---

## 1. Overview

The SLC uses several distinct “message” mechanisms:

1. **Msg table (labeled “Message”)**: Stores **localized text** by **LangCode** and **MsgId**. Used for UI labels, titles, and user-facing messages. **get-Lang-Lbl** and **get-Lang-Msg** look up text in the current language; missing entries are auto-created (English first, then foreign with prefix).
2. **CommHdr / CommDtl**: **Communication header and detail** for sending and receiving messages over the CFS network (e.g. to Inven, Production, other CTS DBs). Used for host sync, goal/order traffic, and an **inbox** (unread messages to the scale/operator). **s-infobar.w** shows a mail icon when unread **CommHdr** exists for the current scale.
3. **ErrorMsgs**: Maps **ReturnCode** (integer) to **ErrorText** (character). Used for standardized error messages (e.g. host update, **update-slc.w**).
4. **Goals.AddMsg1, AddMsg2, AddMsg3**: Optional goal-level text printed on labels via **printlabel-when.i** (tokens **AddMsg1**, **AddMsg2**, **AddMsg3**).
5. **MailID**: Stores **CommSysID** (and EmployeeName, EMailID, etc.) for routing; links operators/locations to communication system IDs.

---

## 2. Msg Table (Localization / “Message”)

### 2.1 Schema

**Source**: `slc.df`, table **Msg** (LABEL "Message", DUMP-NAME "msg")

| Field | Type | Format / Notes |
|-------|------|----------------|
| **LangCode** | character | x(10). Language (e.g. "English", "Spanish"). Part of primary key. |
| **MsgId** | character | x(30), max 60. Message key (e.g. "PrintLabels.Quit", "Login.GlobalKillDateInEffect"). Part of primary key. |
| **Msg** | character | x(256), max 512. The actual text for this language and key. |
| **MsgLen** | integer | Display length limit; **get-Lang-Lbl** returns **SUBSTRING(Msg.Msg, 1, Msg.MsgLen)** so text fits on screen. |

**Index**: **Msg** (unique, primary): **LangCode** ascending, **MsgId** ascending.

**Convention** (from get-lang-msg.i): **MsgId** often follows **ProgramLogicalName.ChildObject** (e.g. **PrintLabels.Quit**, **PrintLabels.Msg.MsgProdLabelFile**). The logical name helps translators identify context.

### 2.2 get-Lang-Lbl and get-Lang-Msg

**Source**: `lib/get-lang-msg.i`

**get-Lang-Lbl**(**ip-LangCode**, **ip-MsgId**) returns character:

- **FIND Msg WHERE Msg.LangCode = ip-LangCode AND Msg.MsgId = ip-MsgId**. If found, returns **SUBSTRING(Msg.Msg, 1, Msg.MsgLen)**.
- If not found: in a transaction, looks up **English** for **ip-MsgId**. If no English row, **CREATE Msg** with LangCode='English', MsgId=ip-MsgId, Msg=ip-MsgId, MsgLen=LENGTH(ip-MsgId). If **ip-LangCode <> 'English'**, **CREATE Msg** for that language with Msg = first 2 chars of LangCode + ip-MsgId (to flag untranslated). Returns the new/found **Msg.Msg** (substring by MsgLen).

So **MsgId** is created on first use; English defaults to the key; other languages get a prefixed key until translated.

**get-Lang-Msg**(**ip-LangCode**, **ip-MsgId**, **ip-values**) returns character:

- Same lookup by LangCode and MsgId. If not available, returns **ip-MsgId**.
- **ip-values** is a **"|"**-separated list. The function replaces **[%1]**, **[%2]**, … in **Msg.Msg** with **ENTRY(1, ip-values, "|")**, **ENTRY(2, ...)**, etc. So messages can have placeholders like "Serial [%1] Not Available." and the caller passes the serial number (or multiple values).

**gLangCode** (session/site) is typically passed as **ip-LangCode** so the UI uses the current language.

### 2.3 Usage

- **Labels and titles**: Buttons, field labels, window titles use **get-Lang-Lbl(gLangCode, "Some.MsgId")**.
- **Messages with substitution**: Confirmations, errors (e.g. "Serial [%1] Not Deleted") use **get-Lang-Msg(gLangCode, "MsgId", "value1|value2")**.
- **Weigh / print**: e.g. **PrintLabels.Msg.MsgProdLabelFile** when product has no label file; **weigh.w** applies a one-time fix replacing "Lable" with "Label" in that **Msg.Msg**.
- **Login**: **s-login.w** seeds **Msg** for **Login.GlobalKillDateInEffect** and **Login.GlobalKillRe-Enter** (English text and MsgLen).

### 2.4 Maintenance and Import/Export

- **viewers/v-translate.w**: SmartViewer over **Msg**. Displays **LangCode**, **MsgLen**, **MsgId**, **Msg**; allows editing **Msg** and **MsgLen** per language. Copy/save logic updates **b-Msg** by **MsgId** and **LangCode**.
- **lib/import-msg-csv.w**: Imports messages from CSV (MsgID, Msg, MsgLen, etc.) into **Msg**; creates or updates by **MsgId** and **LangCode**.
- **system/import.w** and **export.w**: Table list shows **"Message"**; import/export use **msg.d** and table **msg** (DUMP-NAME "msg").
- **lib/add-to-runtime-db.p**: Procedure **2000-Msg** creates/updates **Msg** from arrays **v-MsgID**, **v-Msg**, **v-MsgLangCode**, **v-MsgLen** (seeding runtime messages).

---

## 3. CommHdr and CommDtl (Communications Messaging)

### 3.1 Schema

**CommHdr** (LABEL "Comm Hdr"): “Communications Header used to send messages over the CFS network, including Inven, Production, and CTS DB's.”

| Field | Type | Description |
|-------|------|-------------|
| **CommSeqID** | integer | Primary key (from sequence). |
| **DateRead** | date | Null if not read/processed. |
| **DateSent** | date | Null if not yet sent. |
| **Deleted** | logical | Logical delete. |
| **Priority** | integer | 0 = normal; higher = more important. |
| **TransType** | integer | Transaction ID corresponding to Subject. |
| **SendTo** | character | Recipient (e.g. operator ID, computer ID, or scale ID like "WPL" + scale number). |
| **SentFrom** | character | Sender (operator/computer/origin ID). |
| **Subject** | character | Summary/topic (e.g. "6044", goal subject). |
| **WasRead** | logical | Read/processed. |
| **Type** | character | e.g. "Mail", "DIR" (direction to Inven). |
| **CommSeqIDThread** | integer | Reference back to original message. |
| **ICIProcessed** | logical | Copied/updated by ICI to target DB. |
| **Origin** | character | Application ID / mnemonic (e.g. gICICTSID + scale). |
| **TimeSent** | character | hh:mm:ss sent. |
| **TimeRead** | character | hh:mm:ss read. |

**CommDtl** (LABEL "Comm Detail"): “Child of CommHdr, provides message body; can be multiple details per header; when all details have been added, ReadyToSend should be set=yes.”

| Field | Type | Description |
|-------|------|-------------|
| **CommSeqID** | integer | FK to CommHdr.CommSeqID. |
| **DetailNumber** | integer | Sequence within header. |
| **Body** | character | Message line contents (any data format). |
| **Blob** | raw | Optional binary (e.g. .bmp). |

**Indexes**: CommHdr primary on **CommSeqID**; other indexes on **SendTo**, **SentFrom**, **WasRead**, **Deleted**, **Priority**, **ICIProcessed**, etc. CommDtl primary on **CommSeqID**, **DetailNumber**.

### 3.2 Usage

- **goalcomm.p**: Creates **CommHdr** (Type='Mail', SendTo/SentFrom = operator, Subject = ipSubject) and **CommDtl** (Body = ipText) for internal mail-style messages.
- **msg6044-curr-prodcode-create.p** (and similar): Creates **CommHdr** (Type="DIR", Subject="6044", SendTo="WPL"+scale, Origin=gICICTSID+scale) and **CommDtl** with **Body** for direction messages to Inven.
- **Host / ICI**: CommHdr/CommDtl are used for goal download, serial upload, and other host traffic; **ICIProcessed** marks rows processed by ICI.
- **Inbox (s-infobar.w)**: **ip-SetMailImage** finds **CommHdr** where **SendTo** = **gICICTSID + string(gScaleID,'99')** and **WasRead = no**. If any exist, the mail icon is shown. So messages addressed to the current scale appear as “unread” until marked read.

So **CommHdr** is both the outbound queue (send to host/other apps) and the inbox (messages to this scale/operator). **SendTo** identifies the recipient (scale ID or operator); **Type** and **Subject**/ **TransType** identify the kind of message.

---

## 4. ErrorMsgs

**Source**: `slc.df`, table **ErrorMsgs** (DESCRIPTION "Holds Error Codes", DUMP-NAME "errormsg")

| Field | Type | Description |
|-------|------|-------------|
| **ReturnCode** | integer | Integer error code (primary key). |
| **ErrorText** | character | Error message text. |

**Index**: **RC** (unique, primary) on **ReturnCode**.

**Usage**: **host/update-slc.w** imports **ErrorMsgs** from a temp table and buffer-copies into **ErrorMsgs** (create or update by **ReturnCode**). Used to sync error codes and text from a central source. Other code can look up **ErrorMsgs** by **ReturnCode** to display **ErrorText** to the user or in logs.

---

## 5. Goals.AddMsg1, AddMsg2, AddMsg3

**Source**: `slc.df`, table **Goals**

- **AddMsg1**, **AddMsg2**, **AddMsg3** (character): Optional text fields on the goal. Shown in **goals/v-goals-wo-plc.w** and used on labels when printing for that goal.

**Label usage** (**lib/printlabel.p** via **lib/printlabel-when.i**): When the template contains tokens **AddMsg1**, **AddMsg2**, or **AddMsg3**, **chrReplacementText** is set from **pb-Goals.AddMsg1/2/3** if **pb-Goals** is available and the field is not ?; otherwise **''**. So these are **goal-level message overrides** printed on the case label (e.g. run-specific instructions or notes).

---

## 6. MailID

**Source**: `slc.df`, table **MailID**

| Field | Type | Description |
|-------|------|-------------|
| **EmployeeName** | character | Part of primary key (with EmployeeID). |
| **EmployeeID** | character | Part of primary key. |
| **EMailID** | character | Email identifier. |
| **CommSysID** | character | Communication system ID (e.g. for routing to this operator/location). |
| **Descr** | character | Description. |

**Indexes**: **EmplNameID** (unique, primary) on **EmployeeName**, **EmployeeID**; **CommSysID**; **Descr**.

**Relationship**: **MailID.CommSysID** can match **CommHdr.SendTo** or **SentFrom** (or similar) so that messages are routed to the right operator or mailbox. **s-cfgmenu.w** references MailID maintenance. So **Message** (CommHdr/CommDtl) routing ties to **MailID** via **CommSysID**.

---

## 7. Relationships to Other Concepts

| Concept | Relationship to Message |
|--------|---------------------------|
| **Operator** | **gLangCode** (and thus **Msg** language) may be per operator/site. **CommHdr** SendTo/SentFrom can be operator ID. **MailID** links EmployeeName/EmployeeID to **CommSysID** for messaging. |
| **Goals** | **Goals.AddMsg1/2/3** are message text printed on labels when producing to that goal. Goal download/sync may use **CommHdr**/CommDtl. |
| **Label** | **printlabel-when.i** substitutes **AddMsg1**, **AddMsg2**, **AddMsg3** from **pb-Goals** into label tokens. **Msg** provides localized text for prompts (e.g. missing label file). |
| **Host / Communication** | **CommHdr**/CommDtl are the message layer for host sync (goals, serials, direction messages). **ICIProcessed**, **Type**, **TransType**, **Subject** classify traffic. |
| **Scale / Site** | **s-infobar** shows mail icon when **CommHdr** exists with **SendTo** = gICICTSID + gScaleID and **WasRead** = no. |
| **Config** | **gLangCode** (and any message-related site options) drive which **Msg** rows are used. |
| **Error handling** | **ErrorMsgs** maps **ReturnCode** to **ErrorText**; **update-slc.w** syncs ErrorMsgs from host. |

---

## 8. Business Rules Summary

- **Msg**: **LangCode** + **MsgId** unique. First use of a **MsgId** creates an English row (Msg = MsgId) and, for non-English, a prefixed row. **get-Lang-Lbl** returns text truncated to **MsgLen**.
- **get-Lang-Msg**: Placeholders **[%1]**, **[%2]**, … in **Msg.Msg** are replaced by the corresponding **ip-values** entries (pipe-separated). If no **Msg** row, returns **ip-MsgId**.
- **CommHdr/CommDtl**: One header, one or more details; **CommDtl.CommSeqID** = **CommHdr.CommSeqID**. **SendTo** identifies recipient (scale or operator). **WasRead** drives inbox icon; **ICIProcessed** and **Type** drive host processing.
- **Goals.AddMsg1/2/3**: Optional; only used when **pb-Goals** is available in **PrintLabel** and token is present in template.
- **ErrorMsgs**: **ReturnCode** unique; **ErrorText** is the display text for that code.
- **MailID**: **CommSysID** used for routing; **EmployeeName** + **EmployeeID** unique.

---

## 9. UI and Maintenance

- **Msg**: **v-translate.w** (translate/maintain messages), **import-msg-csv.w** (CSV import), **add-to-runtime-db.p** (seed), import/export “Message” → **msg.d** / table **msg**.
- **CommHdr/CommDtl**: Created by **goalcomm.p**, **msg6044-curr-prodcode-create.p**, and host/ICI logic; read by **s-infobar.w** for inbox icon; cleared in **dbclear** (delCommHdr, delCommDtl).
- **ErrorMsgs**: Maintained/synced via **update-slc.w**; **u-dbclear** can clear.
- **Goals.AddMsg1/2/3**: Edited in **v-goals-wo-plc.w** (and other goal maintenance).
- **MailID**: Config menu has MailID maintenance (btnMailID); import/export include MailID.

---

## 10. Redevelopment Notes

- **Msg**: Implement table (LangCode, MsgId, Msg, MsgLen) with unique (LangCode, MsgId). Implement **get-Lang-Lbl** (lookup + auto-create English/foreign) and **get-Lang-Msg** (lookup + **[%n]** replacement from pipe-separated values). Use **gLangCode** (or equivalent) as current language.
- **CommHdr/CommDtl**: Implement header/detail model; **SendTo** = scale or operator ID for inbox; **WasRead** for unread indicator; **Type**/ **Subject**/ **TransType** for host and internal mail. Replicate **goalcomm.p** and direction (e.g. 6044) creation and **s-infobar** inbox check.
- **ErrorMsgs**: ReturnCode → ErrorText lookup; sync from host if applicable.
- **Goals.AddMsg1/2/3**: Pass **Goals** (or equivalent) into label renderer and substitute AddMsg1/2/3 tokens when goal is present.
- **MailID**: **CommSysID** and employee keys for routing; link to **CommHdr** SendTo/SentFrom where used.
