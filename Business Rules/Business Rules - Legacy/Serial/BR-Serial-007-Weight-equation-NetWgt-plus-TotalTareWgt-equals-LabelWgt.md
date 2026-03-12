---
Risk-Level: P0
Business-Rule-Id: BR-Serial-007
Deprecated: false
---

# Description

For each Serial (and ItemSerial), the sum of NetWgt and TotalTareWgt must equal LabelWgt (or be within tolerance). LabelWgt is the weight printed on the barcode/label; NetWgt is product weight; TotalTareWgt is the sum of tares (BoxTare + ExtraTare + item tares). Data validation checks this equation when creating or validating serial records.

# Source

- [[../Data Models/Serial/Serial - legacy]] — Weight Rules (Tare Calculation), Data Validation Checklist, Example 2 (Validation: NetWgt + TotalTareWgt = LabelWgt)
- Expected: NetWgt + TotalTareWgt = LabelWgt (line 202); "LabelWgt = (NetWgt + Tares) should = LabelWgt (or within tolerance)" (line 178)

# Impacted Systems

Serial table, ItemSerial table, createserial.p, createitemserial.p, weight validation and label weight calculation (Set-Label-Wgt), data validation and reporting (e.g. variance/error when ABS(LabelWgt - NetWgt) exceeds threshold).

# Traceability

- [[../Data Models/0.Legacy Database Schema]] — Serial (LabelWgt, NetWgt, TotalTareWgt, BoxTare, ExtraTare)
- Validation checklist: "NetWgt + TotalTareWgt = LabelWgt"; TotalTareWgt = BoxTare + ExtraTare + ItemTares

# Assertions

- NetWgt + TotalTareWgt = LabelWgt (or within configured tolerance).
- TotalTareWgt = BoxTare + ExtraTare + item tares.
- When validating or reporting, deviation (e.g. ABS(LabelWgt - NetWgt) or comparison to expected sum) may trigger warning or error.

# Related Information

