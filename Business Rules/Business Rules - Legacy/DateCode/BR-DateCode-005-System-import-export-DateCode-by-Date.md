---
Risk-Level: P0
Business-Rule-Id: BR-DateCode-005
Deprecated: false
---

# Description

DateCode is included in the system export and import. Export writes the DateCode table to a file when "Date Codes" is selected from the export table list. Import uses a temp-table TDateCode like DateCode and matches existing rows by Date: KeyPhrase is "DateCode.Date = TDateCode.Date". The import file name is datecode.d. The counter vDateCode tracks the number of DateCode records imported for the result message.

# Source

- progress-SLC/system/export.w: "Date Codes" in export table list; WHEN 'Date Codes' export with TableName DateCode
- progress-SLC/system/import.w: TDateCode like DateCode; vDateCode counter; WHEN 'Date Codes' import from datecode.d, KeyPhrase DateCode.Date = TDateCode.Date

# Impacted Systems

DateCode table, system/export.w, system/import.w, datecode.d.

# Traceability

- Date is the primary key of DateCode; import uses it to update or insert.
- Dump name in slc.df is "datecode".

# Assertions

- Export dumps DateCode rows to the configured file for the Date Codes table.
- Import matches on Date to update existing or create new DateCode rows.

# Related Information

- [[BR-DateCode-001-Date-unique]]
