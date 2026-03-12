---
Risk-Level: P0
Business-Rule-Id: BR-UserBar-002
Deprecated: false
---

# Description

UBDetail has composite primary key (UBName, UBDispLoc) unique (index UBDetIdx). UBName links to UBHeader.UBName (which userbar this detail belongs to). UBDispLoc is the display position (1-based) in the userbar string; details are processed in UBDispLoc order. Each UBDetail row contributes one character to the userbar; the value comes from UBField logic and UBFieldCharLoc (and sometimes UBLength for checksum). UBDetail.UBName is a foreign key to UBHeader; userbar.p finds UBHeader by UBName then iterates FOR EACH UBDetail WHERE UBDetail.UBName = UBHeader.UBName ordered by UBDispLoc.

# Source

- [[../Data Models/UserBar/UserBar - legacy]] — §3 UBDetail Table, §5 userbar.p logic
- [[../Data Models/0.Legacy Database Schema]] — UBDetail (UBName, UBDispLoc, UBField, UBFieldCharLoc, UBLength), index UBDetIdx yes yes UBName, UBDispLoc
- slc.df ubdetail

# Impacted Systems

UBDetail table, UBHeader, lib/userbar.p, label tokens USERBARA–Z.

# Traceability

- One character per UBDetail row; v-userbar = v-userbar + v-char each iteration; UBField maps to SERIAL, PACK_DATE, LOT, ShiftProdCodeCount, LITERAL, etc. from CaseLabelSource, tt-Product, Modify, Goals, Totals, CrossRef

# Assertions

- (UBName, UBDispLoc) is unique.
- UBDetail.UBName must reference an existing UBHeader.UBName.
- UBDispLoc is 1-based; one row per position 1 to UBHeader.UBLength.
- UBField is source field name (e.g. SERIAL, PACK_DATE, LOT); userbar.p CASE CAPS(UBField) maps to value from Serial, Product, Modify, Goals, Totals, CrossRef, or literals.

# Related Information

