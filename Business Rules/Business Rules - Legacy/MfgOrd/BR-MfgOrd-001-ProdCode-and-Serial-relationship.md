---
Risk-Level: P0
Business-Rule-Id: BR-MfgOrd-001
Deprecated: false
---

# Description

MfgOrd table stores MfgOrdDate, MfgOrdNum, ProdCode, MfgOrdStatus. ProdCode is integer format 99999 (same range as Product: 00001–99999). Serial table has MfgOrdNum (int64) linking a printed case to a manufacturing order. MfgOrd identifies manufacturing orders by order number and product; Serial and Goals can reference MfgOrdNum for traceability.

# Source

- [[../Data Models/0.Legacy Database Schema]] — MfgOrd (MfgOrdDate, MfgOrdNum, ProdCode, MfgOrdStatus); Serial.MfgOrdNum (data/Serial-MfgOrdNum.df, data/delta-mfgordnum.df)
- data/MfgOrd.df, data/delta-mfgordnum.df

# Impacted Systems

MfgOrd table, Serial table (MfgOrdNum), Product (ProdCode range), Goals/order flows.

# Traceability

- MfgOrd.ProdCode aligns with Product.ProductCode range; Serial.MfgOrdNum references manufacturing order

# Assertions

- MfgOrd.ProdCode in range 00001–99999 (product code).
- Serial.MfgOrdNum links serial to MfgOrd when produced against a manufacturing order.
- MfgOrd has no unique index documented in schema; application may enforce MfgOrdNum or (MfgOrdDate, MfgOrdNum) uniqueness per implementation.

# Related Information

