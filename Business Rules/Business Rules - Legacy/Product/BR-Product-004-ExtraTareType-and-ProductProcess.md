---
Risk-Level: P0
Business-Rule-Id: BR-Product-004
Deprecated: false
---

# Description

Product must exist before a weigh/print session can use it; tt-Product is built from Product (and family overrides from CrossRef). ProductProcess must exist for the product (or defaults) so that process steps are available. ExtraTareType must be N (None), W (Weight), or P (Percent).

# Source

- [[../Data Models/Product/Product - legacy]] — Business Rules Summary, tt-Product, ProductProcess link
- [[../Data Models/0.Legacy Database Schema]] — Product.ExtraTareType, ProductProcess

# Impacted Systems

Product table, ProductProcess, weigh-create-tt-product.p, Modify (TareToUse when ExtraTareType <> 'N').

# Traceability

- Product.ExtraTareType VALEXP N|W|P; Modify.TareToUse used when Product.ExtraTareType <> 'N'

# Assertions

- Product row must exist for ProductCode used in session.
- ProductProcess must have at least one row for the product (or default product) for steps to be available.
- ExtraTareType in (N, W, P).

# Related Information

