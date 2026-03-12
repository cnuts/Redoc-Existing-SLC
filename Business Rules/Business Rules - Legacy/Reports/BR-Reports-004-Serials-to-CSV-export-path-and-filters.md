---
Risk-Level: P1
Business-Rule-Id: BR-Reports-004
Deprecated: false
---

# Description

The Serials to CSV report (reports/r-serialstocsv.w) exports Serial records to a CSV file whose path is built as: temp\serials-[StartDate MMDDYY]-to-[EndDate MMDDYY]-shift-[all or shift number].csv. Date parts come from fiStartDate and fiEndDate (substring 1,2 / 4,2 / 7,2 for month, day, year). Shift suffix is "all" when rsAllShifts is true, else the numeric fi-Shift formatted as '99'. Only serials matching the following are exported: (1) Date range on either PackDate or PrintDate (rsPackDate: if true, Serial.PackDate >= fiStartDate and <= fiEndDate; else Serial.PrintDate in that range). (2) If not "All Shifts", Serial.Shift = INT(fi-Shift). (3) Serial.Reprinted = false (not serial.Reprinted). Records are sorted BREAK BY serial.productcode. The summary display uses temp/CTS-Serials.txt.

# Source

- progress-SLC/reports/r-serialstocsv.w — vDestination = 'temp\serials-' + substring(string(fiStartDate),1,2)+substring(...,4,2)+substring(...,7,2) + '-to-' + same for fiEndDate + '-shift-' + (if rsAllShifts then 'all' else string(fi-shift,'99')) + '.csv'; output to value(vDestination). for each serial no-lock where (rsPackDate then PackDate range else PrintDate range) and (rsAllShifts or serial.Shift = int(fi-shift)) and not serial.Reprinted break by serial.productcode. output to "temp/CTS-Serials.txt" for run ipSetRptHeadings.

# Impacted Systems

reports/r-serialstocsv.w, Serial table, temp\serials-*.csv, temp/CTS-Serials.txt, fiStartDate, fiEndDate, rsPackDate, rsAllShifts, fi-Shift.

# Traceability

- [[../Data Models/Reports/Serials to CSV Report -legacy]] — Export File Location, Data Source, Filtering
- [[BR-Serial-015-No-Totals-Goals-update-when-serial-reprinted]] — Reprinted excluded from totals; also excluded from CSV export

# Assertions

- CSV file path: temp\serials-[StartDate]-to-[EndDate]-shift-[all|nn].csv.
- Filter: PackDate or PrintDate in [StartDate, EndDate]; optional Shift; exclude Reprinted serials.
- Export order: break by serial.productcode.

# Related Information

