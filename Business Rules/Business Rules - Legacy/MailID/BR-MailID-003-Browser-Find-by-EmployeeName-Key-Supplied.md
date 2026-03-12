---
Risk-Level: P0
Business-Rule-Id: BR-MailID-003
Deprecated: false
---

# Description

The MailID browser (b-mailid.w) lists MailID rows and supports key-based repositioning. It supplies Keys-Supplied = "CommSysID,EMailID" for downstream consumers. On open-query (local-open-query), it uses Key-Value: it finds the first MailID where EmployeeName >= Key-Value (vKeyValue); if none is found it finds the last MailID and repositions there. The MailID container (mailid.w) Find button sets Key-Value to the fiEmployeeName screen value and reopens the browser query, so Find is by EmployeeName (greater-than-or-equal repositioning).

# Source

- progress-SLC/comm/b-mailid.w: Keys-Supplied = "CommSysID,EMailID"; local-open-query — run GetKey (Key-Name, Key-Value); FIND first b-MailID WHERE b-MailID.EmployeeName >= vKeyValue no-lock no-error; IF NOT AVAIL b-MailID THEN FIND last b-MailID no-lock no-error; IF AVAIL b-MailID THEN REPOSITION browse to rowid(b-MailID), set-repositioned-row, APPLY value-changed. send-key: sndkycas "CommSysID" "MailID" "CommSysID", "EMailID" "MailID" "EMailID"
- progress-SLC/comm/mailid.w: btnFind ON CHOOSE: run SetKey, fiEmployeeName:screen-value = ''; SetKey: set-attribute-list Key-Value = fiEmployeeName:screen-value, dispatch open-query in h_b-mailid

# Impacted Systems

b-mailid.w, mailid.w, MailID table, CommSysID/EMailID key supply for communication UIs.

# Traceability

- Repositioning is by EmployeeName (>=); empty Key-Value repositions to first row (all MailID satisfy >= '').
- After Find, fiEmployeeName is cleared in mailid.w so the next Find can be entered fresh.

# Assertions

- Key-Value is used as the minimum EmployeeName for the first matching row; if no row has EmployeeName >= Key-Value, the browse repositions to the last row.
- send-key returns MailID.CommSysID or MailID.EMailID when requested by key name.

# Related Information

- [[BR-MailID-001-EmployeeName-EmployeeID-unique-CommSysID-routing]]
