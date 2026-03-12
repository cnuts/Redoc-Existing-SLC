---
Risk-Level: P0
Business-Rule-Id: BR-Serial-012
Deprecated: false
---

# Description

Before updating Totals (and Goals), lib/updatetotals.p validates the serial buffer. If pb-serial.ProductCode LE 0 OR pb-serial.ProductCode GT 99999, return "Product Code out of range". If pb-serial.LabelWgt LE 0 OR pb-serial.LabelWgt GT 9999.99, return "Label Weight out of range". If pb-serial.NetWgt LE 0 OR pb-serial.NetWgt GT 9999.99, return "Net Weight out of range". When any of these conditions is true, the procedure assigns the error string to v-RetVal and no totals update is performed. Caller should check return-value after Run UpdateTotals; empty return-value means no errors.

# Source

- progress-SLC/lib/updatetotals.p — procedure UpdateTotals, 050-Process-Serial: ASSIGN v-RetVal = ( IF pb-serial.ProductCode LE 0 OR pb-serial.ProductCode GT 99999 THEN "Product Code out of range" ELSE IF pb-serial.LabelWgt LE 0 OR pb-serial.LabelWgt GT 9999.99 THEN "Label Weight out of range" ELSE IF pb-serial.NetWgt LE 0 OR pb-serial.NetWgt GT 9999.99 THEN "Net Weight out of range" ELSE v-RetVal )

# Impacted Systems

Serial buffer, Totals table, Goals table, lib/updatetotals.p, callers (e.g. createserial flow via btnPrint of s-printlbl.w).

# Traceability

- [[../Data Models/Serial/Serial - legacy]] — LabelWgt, NetWgt; ProductCode range
- [[../Data Models/0.Legacy Database Schema]] — Serial, Totals

# Assertions

- For UpdateTotals to proceed: ProductCode must be > 0 and <= 99999.
- LabelWgt must be > 0 and <= 9999.99.
- NetWgt must be > 0 and <= 9999.99.
- When any check fails, v-RetVal (po-RetVal) is set to the corresponding error string and no totals update is done.

# Related Information

