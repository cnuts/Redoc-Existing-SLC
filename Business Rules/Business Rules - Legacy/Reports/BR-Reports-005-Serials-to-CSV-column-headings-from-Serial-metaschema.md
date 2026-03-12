---
Risk-Level: P1
Business-Rule-Id: BR-Reports-005
Deprecated: false
---

# Description

The Serials to CSV report writes column headings in the CSV file from the Progress metaschema for the Serial table. Before exporting data, it loops FOR EACH _file NO-LOCK WHERE _file-name = 'serial' and FOR EACH _field OF _file NO-LOCK BY _order, appending _field._field-name and a comma to vColumnHeadings. The first line of the CSV is this comma-delimited list of field names, so the export column order matches the Serial table definition order. This ensures that if fields are added to the Serial table, the CSV header and export statement (EXPORT DELIMITER ',' serial) stay aligned without code change.

# Source

- progress-SLC/reports/r-serialstocsv.w — for each _file no-lock where _file-name = 'serial': for each _field of _file no-lock by _order: vColumnHeadings = vColumnHeadings + _field._field-name + ','. put unformatted vColumnHeadings skip. export delimiter ',' serial.

# Impacted Systems

reports/r-serialstocsv.w, Serial table, _file/_field metaschema, CSV export format.

# Traceability

- [[../Data Models/0.Legacy Database Schema]] — Serial table field list
- [[BR-Reports-004-Serials-to-CSV-export-path-and-filters]] — same report

# Assertions

- CSV first line is the list of Serial field names from _field by _order.
- Data rows are exported with EXPORT DELIMITER ',' serial, so column order matches the schema.

# Related Information

