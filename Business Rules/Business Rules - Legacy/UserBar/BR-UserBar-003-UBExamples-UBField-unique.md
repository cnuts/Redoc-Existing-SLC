---
Risk-Level: P0
Business-Rule-Id: BR-UserBar-003
Deprecated: false
---

# Description

UBExamples table has UBField as primary key (unique, index UBExampIdx). UBField matches UBDetail.UBField (the source field name). UBExample is character x(48)—example value or format (e.g. "YYYYMMDD" for Packdate, "0123456789" for Literal). Used in the UI to guide setup; not used by userbar.p at runtime. Records are seeded in lib/add-to-runtime-db.p (e.g. 7500-Build-UBExamples); there is no UI to add UBExamples; they are created in code.

# Source

- [[../Data Models/UserBar/UserBar - legacy]] — §4 UBExamples Table
- [[../Data Models/0.Legacy Database Schema]] — UBExamples (UBField, UBExample), index UBExampIdx yes yes UBField
- slc.df ubexampl

# Impacted Systems

UBExamples table, UBDetail.UBField (same name space), add-to-runtime-db.p, UserBar setup UI (documentation/examples for each UBField).

# Traceability

- UBField in UBExamples documents valid UBField names and example formats for UBDetail configuration
- Not used at runtime by userbar.p

# Assertions

- UBField is unique (primary key) in UBExamples.
- UBExample provides example value or format for setup guidance only.
- UBExamples are created in code (add-to-runtime-db.p), not via end-user UI.

# Related Information

