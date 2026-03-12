---
Risk-Level: P0
Business-Rule-Id: BR-Product-001
Deprecated: false
---

# Description

ProductCode must be in the range 00001–99999. ProductCode is the primary key of the Product table and must be unique. The same range is enforced on Serial, ItemSerial, ProductProcess, Modify, Totals, Goals (ProdCode), Locals, and PalletDtl where they store product.

# Source

- [[../Data Models/Product/Product - legacy]] — Schema (VALEXP ProductCode > 0 AND ProductCode <= 99999), Business Rules Summary
- [[../Data Models/0.Legacy Database Schema]] — Product (ProductCode), index prod unique on ProductCode

# Impacted Systems

Product table, Serial, ItemSerial, ProductProcess, Modify, Totals, Goals, Locals, PalletDtl.

# Traceability

- slc.df Product table; VALEXP/VALMSG "Product Code must be in the range 00001-99999."

# Assertions

- ProductCode integer, format 99999, mandatory.
- ProductCode > 0 AND ProductCode <= 99999.
- ProductCode is unique (primary key).

# Related Information

