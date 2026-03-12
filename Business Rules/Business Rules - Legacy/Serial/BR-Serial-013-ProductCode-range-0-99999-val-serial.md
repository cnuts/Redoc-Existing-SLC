---
Risk-Level: P0
Business-Rule-Id: BR-Serial-013
Deprecated: false
---

# Description

In lib/val-serial.p, ProductCode is validated: if int(ip-Field-Val) < 0 or int(ip-Field-Val) > 99999 then op-error = "Val.Serial.ProdCode.OutOfLimit". So val-serial allows ProductCode in the range 0 to 99999 (inclusive). Note: lib/updatetotals.p requires ProductCode > 0 and <= 99999 for updating totals (see [[BR-Serial-012-UpdateTotals-range-checks-ProductCode-LabelWgt-NetWgt]]).

# Source

- progress-SLC/lib/val-serial.p — case "ProductCode": if int(ip-Field-Val) < 0 or int(ip-Field-Val) > 99999 then op-error = "Val.Serial.ProdCode.OutOfLimit"

# Impacted Systems

Serial table, lib/val-serial.p, Product table (ProductCode).

# Traceability

- [[../Data Models/0.Legacy Database Schema]] — Serial.ProductCode, Product.ProductCode
- [[BR-Serial-012-UpdateTotals-range-checks-ProductCode-LabelWgt-NetWgt]]

# Assertions

- In val-serial.p, ProductCode must be >= 0 and <= 99999.
- When validation fails, op-error is set to "Val.Serial.ProdCode.OutOfLimit".

# Related Information

