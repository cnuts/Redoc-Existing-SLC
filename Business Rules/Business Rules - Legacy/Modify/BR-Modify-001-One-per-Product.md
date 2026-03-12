---
Risk-Level: P0
Business-Rule-Id: BR-Modify-001
Deprecated: false
---

# Description

There is at most one Modify row per ProductCode. Modify.ProductCode is the primary key (unique). Modify holds runtime overrides (TareToUse, PackDateOffset, KillDateOffset, VarOffset, Price, Lot, CustName, OrdNum, ModText1, ModText2, CasesPerPallet, PalletNum, PalletTracking, PdnOrdLineNum) applied on top of Product when creating Serial and labels.

# Source

- [[../Data Models/Modify/Modify - legacy]] — §1 Overview, §2 Schema, §12 Business Rules Summary
- [[../Data Models/0.Legacy Database Schema]] — Modify table, index Modify on ProductCode

# Impacted Systems

Modify table, tt-Modify, createserial.p, weigh-create-modify.p, s-modify.w.

# Traceability

- "Data that can be changed by product code in the modify screen. The current values are saved by product code."

# Assertions

- ProductCode 00001–99999, unique (one Modify per product).
- ModText1, ModText2 length ≤ 20 (VALEXP). ModText1 can store NVP data (e.g. ZuluDept).
- PackDateOffset, KillDateOffset, VarOffset passed into CreateSerial for pack/kill/variable dates.

# Related Information

