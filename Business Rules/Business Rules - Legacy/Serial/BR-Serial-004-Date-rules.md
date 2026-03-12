---
Risk-Level: P0
Business-Rule-Id: BR-Serial-004
Deprecated: false
---

# Description

PrintDate and PackDate must not be in the future (can't print in the past per validation checklist: PrintDate ≤ TODAY, PackDate ≤ TODAY). KillDate is derived from PackDate and product shelf life. SellByDate = KillDate + SellByOffset. Offsets (SellByOffset, VarOffset, etc.) modify dates at label print time.

# Source

- [[../Data Models/Serial/Serial - legacy]] — Date Rules, Data Validation Checklist
- [[../Data Models/Modify/Modify - legacy]] — PackDateOffset, KillDateOffset, VarOffset

# Impacted Systems

Serial creation (createserial.p), Modify/Goals date overrides, label and barcode date logic.

# Traceability

- [[../Data Models/0.Legacy Database Schema]] — Serial (PrintDate, PackDate, KillDate)
- CreateSerial receives piPackDateOffset, piKillDateOffset, piVariableOffset from tt-Modify

# Assertions

- PrintDate set to TODAY at creation.
- PrintTime set to SECONDS(MTIME) for per-second precision.
- KillDate = PackDate + product shelf life (or equivalent).
- SellByDate = KillDate + SellByOffset (calculated at label print time).
- PackDate and KillDate ≤ TODAY per validation checklist.

# Related Information

