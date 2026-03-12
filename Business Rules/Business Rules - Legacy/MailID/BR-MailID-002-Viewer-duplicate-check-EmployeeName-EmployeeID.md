---
Risk-Level: P0
Business-Rule-Id: BR-MailID-002
Deprecated: false
---

# Description

The MailID viewer (v-mailid.w) enforces uniqueness of (EmployeeName, EmployeeID) on create. When saving a new record (ADM-NEW-RECORD = yes), before dispatching update-record it finds an existing MailID with the same EmployeeName and EmployeeID as the screen values; if one exists it shows the message "MailID.EmployeeName/ID Already Exists; Please Change MailID.EmployeeName/ID", applies entry to EmployeeName, and returns without saving. Add and Copy operations prompt the user to set EmployeeName and ID (and other values) so the key is populated before save.

# Source

- progress-SLC/comm/v-mailid.w: local-update-record — RUN get-attribute "ADM-NEW-RECORD"; if 'yes' then FIND first b-MailID WHERE b-MailID.EmployeeName = MailID.EmployeeName:screen-value AND b-MailID.EmployeeID = MailID.EmployeeID:screen-value no-lock no-error; IF AVAIL b-MailID then d-tsmsgbox "Create New MailID" "EmployeeName/ID Already Exists... Please Change...", APPLY 'entry' to MailID.EmployeeName, RETURN; else RUN dispatch update-record. local-add-record: d-tsmsgbox "Please set MailID.EmployeeName, ID and other values". local-copy-record: "Copy in progress: Please modify MailID.EmployeeName & ID"

# Impacted Systems

MailID table, v-mailid.w (comm), ADM add/copy/update flow, d-tsmsgbox.

# Traceability

- Duplicate check uses (EmployeeName, EmployeeID) matching the primary unique index EmplNameID (see [[BR-MailID-001-EmployeeName-EmployeeID-unique-CommSysID-routing]]).
- User is directed to change EmployeeName (and implicitly EmployeeID) when duplicate is detected.

# Assertions

- Only when ADM-NEW-RECORD = yes is the duplicate check performed; updates to existing records are not blocked by this check.
- Add and Copy show informational messages reminding the user to set the key fields.

# Related Information

- [[BR-MailID-001-EmployeeName-EmployeeID-unique-CommSysID-routing]]
