---
Risk-Level: P0
Business-Rule-Id: BR-DateCode-004
Deprecated: false
---

# Description

The Find Date Code dialog (d-finddatecode.w) looks up a DateCode by the user-entered date (fil-date) and returns poRowidDateCode and poCancel. On OK, it finds the first DateCode where DateCode.Date >= fil-date. If no row is found, it then finds the last DateCode where DateCode.Date < fil-date. If that also fails, it shows "No Dates Exist, create some before finding", sets poCancel = true, and returns. If the "last before" row exists, it shows "&lt;fil-date&gt; does not exist, found previous", sets poCancel = false and poRowidDateCode = rowid(DateCode), and returns. If the first find (>= fil-date) succeeded but DateCode.Date <> fil-date, it shows "&lt;fil-date&gt; does not exist, found next". In all cases where a row was found, poRowidDateCode is set to that row's rowid and poCancel = false. Thus the lookup is greater-than-or-equal; the user is informed when the exact date is missing (found next or found previous) or when no DateCode rows exist.

# Source

- progress-SLC/dialogs/d-finddatecode.w: assign fil-date; FIND first datecode WHERE datecode.date >= fil-date no-lock no-error. IF NOT AVAILABLE then FIND last datecode WHERE datecode.date < fil-date no-error; IF NOT AVAIL then d-tsmsgbox "FIND DATE" "No Dates Exist, create some before finding", poCancel = true, return; ELSE d-tsmsgbox "FIND DATE" string(fil-date) + " does not exist, found previous", poCancel = false, poRowidDateCode = rowid(DateCode), return. IF datecode.date <> fil-date then d-tsmsgbox "FIND DATE" string(fil-date) + " does not exist, found next". poRowidDateCode = rowid(DateCode), poCancel = false.

# Impacted Systems

DateCode table, d-finddatecode.w, s-datecodes.w or caller that uses poRowidDateCode.

# Traceability

- GTE lookup pattern; "found next" and "found previous" messages match other find dialogs (e.g. Product, Operator, Item).
- poRowidDateCode positions the browser/viewer on the chosen or nearest DateCode row.

# Assertions

- When no DateCode rows exist, the only message is "No Dates Exist, create some before finding" and poCancel = true.
- When an exact match exists, no "found next" message is shown; poRowidDateCode is set to that row.

# Related Information

- [[BR-DateCode-001-Date-unique]]
