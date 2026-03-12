---
Risk-Level: P0
Business-Rule-Id: BR-Serial-003
Deprecated: false
---

# Description

LabelWgt must not exceed product maximum weight (MaxWgt). NetWgt must meet product minimum weight (MinWgt). Total tare is computed as BoxTare + ExtraTare + ItemTares. Weight units (WgtUnits) must be "LB" or "KG". If trim(ip-Field-Val) is neither "LB" nor "KG", lib/val-serial.p returns error "Val.Serial.WgtUnits.WrongUnits".

# Source

- [[../Data Models/Serial/Serial - legacy]] — Weight Rules
- [[../Data Models/Product/Product - legacy]] — MinWgt, MaxWgt, BoxTare, ExtraTareType
- Set-Label-Wgt logic (print/set-label-wgt.p)
- progress-SLC/lib/val-serial.p — case "WgtUnits": if not(trim(ip-Field-Val) = "LB" or trim(ip-Field-Val) = "KG") then op-error = "Val.Serial.WgtUnits.WrongUnits"

# Impacted Systems

Serial table, ItemSerial table, lib/val-serial.p, Serial creation, label print, Product/tt-Product weight validation.

# Traceability

- [[../Data Models/0.Legacy Database Schema]] — Serial (LabelWgt, NetWgt, BoxTare, ExtraTare, TotalTareWgt)
- Data validation checklist: LabelWgt within Product.MinWgt to Product.MaxWgt

# Assertions

- LabelWgt ≤ Product.MaxWgt (or tt-Product).
- NetWgt ≥ Product.MinWgt.
- If |NetWgt − Expected| > Tolerance, operators may be warned (variance check).
- TotalTareWgt = BoxTare + ExtraTare + ItemTares.
- WgtUnits must be "LB" or "KG"; when validation fails, op-error = "Val.Serial.WgtUnits.WrongUnits".

# Related Information

