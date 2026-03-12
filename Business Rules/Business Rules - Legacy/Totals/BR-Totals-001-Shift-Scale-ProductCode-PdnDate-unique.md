---
Risk-Level: P0
Business-Rule-Id: BR-Totals-001
Deprecated: false
---

# Description

Totals table has primary unique index ShiftScaleProdDate on (Shift, Scale, ProductCode, PdnDate). One row per combination of shift, scale, product, and production date. Totals hold LocalCount, LocalLabelWgt, LocalNetWgt, LocalTareWgt, TotalCount, TotalLabelWgt, TotalNetWgt, PdnDate, and cancellation counts (LocalCancelCount, LocalCancelLabelWgt, LocalCancelNetWgt). Aggregates are updated when serials are added or deleted for that product (updatetotals.p, updatetotals2.p).

# Source

- [[../Data Models/0.Legacy Database Schema]] — Totals table, index ShiftScaleProdDate (yes, yes)
- [[../Data Models/Serial/Serial - legacy]] — UpdateTotals; Totals keyed by Shift, Scale, PdnDate, ProductCode
- [[../Data Models/Goal/Goal - Legacy]] — updatetotals.p, updatetotals2.p update Totals and Goals

# Impacted Systems

Totals table, Serial/ItemSerial add/delete, lib/updatetotals.p, lib/updatetotals2.p, production reporting.

# Traceability

- Totals: ProductCode, Scale, Shift, PdnDate, LocalCount, LocalLabelWgt, LocalNetWgt, LocalTareWgt, TotalCount, TotalLabelWgt, TotalNetWgt, LocalCancelCount, LocalCancelLabelWgt, LocalCancelNetWgt
- PdnDateScShProduct (no, yes) on PdnDate, Scale, Shift, ProductCode

# Assertions

- (Shift, Scale, ProductCode, PdnDate) unique.
- Scale and ProductCode in same ranges as Serial (ScaleNumber, ProductCode).
- Shift in (1, 2, 3). Totals updated in same transaction as Serial create/delete when GoalID or product/scale/shift/date apply.

# Related Information

