---
Risk-Level: P0
Business-Rule-Id: BR-MailID-004
Deprecated: false
---

# Description

In the communications console (commcons7.w), when the user mode is set to email or to "all messages", the list of selectable users (sel-users) is populated from the MailID table. For every MailID row where EMailID is not blank, MailID.CommSysID is added to sel-users. Thus only MailID entries with a non-empty EMailID are offered as message recipients (or for the email/all-messages view). Processes (v-ICICTSID + string 1–99) are added when user mode is processes or all messages.

# Source

- progress-SLC/comm/commcons7.w: IF rs-UserMode:screen-value = vMsgEmail OR rs-UserMode:screen-value = vMsgAllMessages THEN DO: FOR EACH MailID WHERE MailID.EMailID <> '' no-lock: sel-users:add-last(MailID.CommSysID). END. END. Then when processes or all messages, add process IDs to sel-users.

# Impacted Systems

commcons7.w, MailID table (EMailID, CommSysID), sel-users list, message/email routing.

# Traceability

- CommSysID is the value used for routing (e.g. SendTo/SentFrom); only rows with EMailID <> '' are included so that only "email-capable" recipients appear.
- Filtering on EMailID <> '' ensures MailID rows without an email address are not selectable in email/all-messages mode.

# Assertions

- Every MailID with non-blank EMailID has its CommSysID added to sel-users exactly once (FOR EACH, add-last).
- When user mode is neither email nor all messages, this MailID loop is not run; processes may still be added.

# Related Information

- [[BR-MailID-001-EmployeeName-EmployeeID-unique-CommSysID-routing]]
- [[BR-Comm-001-CommDtl-CommHdr-ReadyToSend]]
