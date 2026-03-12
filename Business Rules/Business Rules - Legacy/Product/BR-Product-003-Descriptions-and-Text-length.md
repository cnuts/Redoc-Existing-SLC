---
Risk-Level: P0
Business-Rule-Id: BR-Product-003
Deprecated: false
---

# Description

Desc1 and Desc2 length ≤ 32 characters. ShortDesc1, ShortDesc2, ShortDesc3 length ≤ 8. Text1 through Text10 length ≤ 64 characters each.

# Source

- [[../Data Models/Product/Product - legacy]] — Schema (X(32), X(8), X(64)), Business Rules Summary
- [[../Data Models/0.Legacy Database Schema]] — Product field formats

# Impacted Systems

Product table, labels, UserBar, tt-Product, printlabel.p.

# Traceability

- slc.df Product: Desc1/Desc2 X(32), ShortDesc1–3 X(8), Text1–10 X(64)

# Assertions

- Desc1, Desc2: length ≤ 32.
- ShortDesc1, ShortDesc2, ShortDesc3: length ≤ 8.
- Text1–Text10: length ≤ 64.

# Related Information

