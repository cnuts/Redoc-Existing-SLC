---
Risk-Level: P0
Business-Rule-Id: BR-Totals-004
Deprecated: false
---

# Description

When the operation is Delete (chrAddOrDelete = "D") and a Totals row exists for (Shift, Scale, ProductCode, PdnDate), lib/updatetotals.p and lib/updatetotals2.p either decrement the row or delete it. If Totals.LocalCount - 1 > 0: decrement LocalCount by 1; subtract pb-serial.LabelWgt from LocalLabelWgt, pb-serial.NetWgt from LocalNetWgt, (BoxTare + ExtraTare) from LocalTareWgt, with each result floored at zero (if subtraction would go negative, assign 0). If Totals.LocalCount - 1 <= 0: DELETE the Totals row and leave (v-RetVal/po-RetVal set to "" in updatetotals.p).

# Source

- progress-SLC/lib/updatetotals.p — 500-delete: IF Totals.LocalCount - 1 GT 0 then ASSIGN Totals.LocalCount = Totals.LocalCount - 1, Totals.LocalLabelWgt = (IF Totals.LocalLabelWgt - pb-serial.LabelWgt GE 0 THEN … ELSE 0), same for LocalNetWgt and LocalTareWgt. ELSE DO: DELETE Totals. ASSIGN v-RetVal = "". LEAVE 050-Process-Serial. END.
- progress-SLC/lib/updatetotals2.p — ELSE DO: IF Totals.LocalCount - 1 GT 0 THEN ASSIGN (same logic). ELSE DELETE Totals. END.

# Impacted Systems

Totals table, lib/updatetotals.p, lib/updatetotals2.p, serial cancel/delete flow.

# Traceability

- [[BR-Totals-001-Shift-Scale-ProductCode-PdnDate-unique|BR-Totals-001]]
- [[../Data Models/0.Legacy Database Schema]] — Totals LocalCount, LocalLabelWgt, LocalNetWgt, LocalTareWgt

# Assertions

- On serial delete, LocalCount is decremented by 1 or the Totals row is deleted when count would become 0 or less.
- LocalLabelWgt, LocalNetWgt, LocalTareWgt are never set below zero when decrementing.

# Related Information

