---
Risk-Level: P0
Business-Rule-Id: BR-Locals-001
Deprecated: false
---

# Description

Locals table has composite primary key (LocalCode, PageNbr) unique. LocalCode is integer format 9999; ProductCode is in range 00001–99999 (Product Code); PageNbr is Locals Page Number (format 999). Index lc is primary unique on LocalCode, PageNbr; PageNumber index on PageNbr, LocalCode. Locals are used for display/local code mapping by product (pageheader, page number per local and product).

# Source

- [[../Data Models/0.Legacy Database Schema]] — Locals table (LocalCode, ProductCode, PageNbr, pageheader), indexes lc, PageNumber
- slc.df locals
- [[../Data Models/Product/Product - legacy]] — Locals.ProductCode for display area

# Impacted Systems

Locals table; product display/local pages, v-products.w (Locals and product code checks).

# Traceability

- Primary key (LocalCode, PageNbr); ProductCode FK range 00001–99999; pageheader X(40)

# Assertions

- (LocalCode, PageNbr) is unique.
- ProductCode in range 00001–99999 when present.
- Locals link LocalCode and PageNbr to ProductCode and pageheader for display.

# Related Information

