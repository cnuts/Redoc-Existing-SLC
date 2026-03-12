---
Risk-Level: P1
Business-Rule-Id: BR-Product-009
Deprecated: false
---

# Description

The Find Product dialog (dialogs/d-prodfind.w) sets gProductRowid from a Product lookup by the entered product code (fiProdCode). Procedure ip-find-product: FIND first Product where Product.ProductCode >= fiProdCode no-lock no-error. If a product is found but its ProductCode is not equal to fiProdCode, the user is shown "Product [fiProdCode] Not Found, Found Next". If no product is found (not avail), the procedure finds the last Product. If that exists, the user is shown "Product [fiProdCode] Not Found, Found Previous"; otherwise "No Products Exist". In all cases where a Product buffer is available, gProductRowid is set to rowid(Product), and poCancel = false. So the dialog uses a greater-than-or-equal lookup; when the exact code is missing, it either positions to the next higher product or the last product and informs the user.

# Source

- progress-SLC/dialogs/d-prodfind.w — procedure ip-find-product: assign fiProdCode. FIND first product where product.productcode >= fiProdCode no-lock no-error. if avail and productcode <> fiProdCode: run d-tsmsgbox "Product [fiProdCode] Not Found, Found Next". if not avail: FIND last product no-lock no-error; if avail then msg "Found Previous" else "No Products Exist". gProductRowid = rowid(Product). poCancel = false.

# Impacted Systems

dialogs/d-prodfind.w, Product table, gProductRowid, sobjects/s-products.w (runs dialog).

# Traceability

- [[BR-Product-001-ProductCode-range-unique]] — lookup by ProductCode
- Find Product sets global rowid for product browser/selection

# Assertions

- Lookup is first Product where ProductCode >= entered code.
- If exact match not found but a higher product exists, user is told "Not Found, Found Next".
- If no product >= code exists, last product is used and user is told "Found Previous" or "No Products Exist".
- gProductRowid is set to the found (or last) product's rowid.

# Related Information

