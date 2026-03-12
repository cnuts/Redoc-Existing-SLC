---
Risk-Level: P0
Business-Rule-Id: BR-DateCode-002
Deprecated: false
---

# Description

In lib/formatdatetime.p, when the date format requested is "P" (user date code for internal encoding), the FormatDate procedure finds DateCode where DateCode.Date = date(chrDateTime). If a row is available, chrFormatDate is set to DateCode.Code (the code can be up to 10 characters per schema). If no row is found, chrFormatDate is set to string(date(chrDateTime)). Thus format "P" in label printing uses the DateCode table to substitute a customer-defined code for the date; absence of a DateCode row for that date falls back to the default date string.

# Source

- progress-SLC/lib/formatdatetime.p: when "P" then FIND datecode WHERE datecode.date = date(chrDateTime) no-lock no-error; IF AVAIL datecode THEN chrFormatDate = DateCode.Code; ELSE chrFormatDate = string(date(chrDateTime)). Comment: "added this for use with table datecode so that customers can encode a date for internal purposes. This date can be up to 10 characters"

# Impacted Systems

DateCode table (Date, Code), lib/formatdatetime.p, label printing (s-printlbl.w, printlabel.p when format "P" is used).

# Traceability

- Format "P" is one of the FormatDate options (A, 0, P, Q, R, S, T, u, etc.); invoked from label print when the format specifier is "P".
- Code is character X(10) in schema.

# Assertions

- Lookup is by Date only (primary key).
- When no DateCode row exists for the given date, the result is the default string representation of the date, not an error.

# Related Information

- [[BR-DateCode-001-Date-unique]]
