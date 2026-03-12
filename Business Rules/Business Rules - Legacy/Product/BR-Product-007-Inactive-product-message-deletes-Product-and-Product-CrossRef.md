---
Risk-Level: P0
Business-Rule-Id: BR-Product-007
Deprecated: false
---

# Description

When the inactive-product message is processed (host/msg-inactive-product.p), the message body is read as JSON into temp-table tt-InactiveProduct (field ProdCode). For each tt-InactiveProduct, the procedure finds the Product with that ProductCode (exclusive-lock no-wait). If the Product is available, it deletes the Product. It then finds the CrossRef where Application = "Product" and ID = string(ProdCode,"99999"); if that CrossRef exists, it is deleted. So marking a product inactive via this message removes both the Product record and the associated Product CrossRef record (Application "Product", ID = product code).

# Source

- progress-SLC/host/msg-inactive-product.p — DO: tt-InactiveProduct:READ-JSON from ip-MsgBody. FOR EACH tt-InactiveProduct: FIND Product EXCLUSIVE-LOCK WHERE ProductCode = tt-InactiveProduct.ProdCode NO-ERROR NO-WAIT. IF AVAILABLE Product THEN DELETE Product. FIND CrossRef WHERE Application = "Product" AND ID = STRING(tt-InactiveProduct.ProdCode,"99999") NO-ERROR NO-WAIT. IF AVAILABLE CrossRef THEN DELETE CrossRef.

# Impacted Systems

host/msg-inactive-product.p, Product table, CrossRef table (Application = "Product"), messaging host.

# Traceability

- [[BR-Product-001-ProductCode-range-unique]] — ProductCode range; inactive message uses ProdCode from JSON
- [[BR-CrossRef-001-Application-ID-unique]] — CrossRef Application + ID

# Assertions

- Processing the inactive-product message deletes the Product row for each ProdCode in the message.
- For each deleted Product, the CrossRef row with Application = "Product" and ID = that product code is also deleted when present.

# Related Information

