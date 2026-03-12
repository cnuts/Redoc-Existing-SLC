---
Risk-Level: P0
Business-Rule-Id: BR-MailID-005
Deprecated: false
---

# Description

MailID table is included in system export and import. Export writes MailID rows to the file MailID.d (export.w: when "MailID" is selected, export.i with TableName = MailID, ExportFile = MailID.d). Import reads from MailID.d and matches existing rows by (EmployeeName, EmployeeID): import2.i is invoked with KeyPhrase "MailID.EmployeeName = TMailID.EmployeeName and MailID.EmployeeID = TMailID.EmployeeID", so the key for update/insert resolution is the same as the primary unique index EmplNameID.

# Source

- progress-SLC/system/export.w: WHEN 'MailID' THEN {system/export.i &ExportFile = "MailID.d" &TableName = "MailID"}
- progress-SLC/system/import.w: temp-table tMailID like MailID; WHEN 'MailID' THEN {system/import.i &ImportFile = "MailID.d" &TableName = "MailID" &CounterName = "vMailID"}; import2.i with &KeyPhrase = "MailID.EmployeeName = TMailID.EmployeeName and MailID.EmployeeID = TMailID.EmployeeID"

# Impacted Systems

MailID table, system/export.w, system/import.w, export.i, import.i, import2.i, MailID.d file.

# Traceability

- Export: full table dump to MailID.d.
- Import: records in MailID.d are matched to MailID by (EmployeeName, EmployeeID); counter vMailID tracks count for result message.

# Assertions

- Import key phrase explicitly uses EmployeeName and EmployeeID, matching the primary key of MailID.
- MailID is listed in the export/import table selection lists ("MailID" in LIST-ITEMS).

# Related Information

- [[BR-MailID-001-EmployeeName-EmployeeID-unique-CommSysID-routing]]
