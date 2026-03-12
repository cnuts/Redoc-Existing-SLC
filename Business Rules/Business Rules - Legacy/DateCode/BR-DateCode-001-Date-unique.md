---
Risk-Level: P0
Business-Rule-Id: BR-DateCode-001
Deprecated: false
---

# Description

The DateCode table maps a Date to a Code. Date is the primary key and must be unique (index dateidx unique on Date ascending). Each date has at most one row.

# Source

- [[../Data Models/0.Legacy Database Schema]] — DateCode table (Date, Code), index dateidx yes yes Date ASCENDING
- slc.df datecode

# Impacted Systems

DateCode table; any logic that looks up a code by date.

# Traceability

- DateCode: Date (date), Code (character X(10)); dateidx primary unique

# Assertions

- Date is unique in DateCode.
- One Code value per Date.

# Related Information

