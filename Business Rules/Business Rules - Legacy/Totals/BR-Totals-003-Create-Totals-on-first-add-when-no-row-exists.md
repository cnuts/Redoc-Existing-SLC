---
Risk-Level: P0
Business-Rule-Id: BR-Totals-003
Deprecated: false
---

# Description

When lib/updatetotals.p or lib/updatetotals2.p runs and no Totals row exists for the key (Shift, Scale, ProductCode, PdnDate) from the serial buffer, and the operation is Add (chrAddOrDelete = "A"), the procedure creates a new Totals row. Initial values: Totals.Shift = pb-serial.Shift, Totals.Scale = pb-serial.Scale, Totals.ProductCode = pb-serial.ProductCode, Totals.PdnDate = pb-serial.packdate, Totals.LocalCount = 1, Totals.LocalLabelWgt = pb-serial.LabelWgt, Totals.LocalNetWgt = pb-serial.NetWgt, Totals.LocalTareWgt = pb-serial.BoxTare + pb-serial.ExtraTare. When the operation is Delete and no Totals row exists, no row is created and no update is performed (updatetotals.p leaves the find-totals loop when "not locked" and leaves 050-Process-Serial without creating).

# Source

- progress-SLC/lib/updatetotals.p — IF NOT AVAIL(Totals) THEN 200-Create: DO: IF chrAddOrDelete EQ "A" THEN 300-Add: DO: CREATE Totals. ASSIGN Totals.Shift = pb-serial.Shift, Totals.Scale = pb-serial.Scale, Totals.ProductCode = pb-serial.ProductCode, Totals.LocalCount = 1, Totals.PdnDate = pb-serial.packdate, Totals.LocalLabelWgt = pb-serial.LabelWgt, Totals.LocalNetWgt = pb-serial.NetWgt, Totals.LocalTareWgt = pb-serial.BoxTare + pb-serial.ExtraTare, v-RetVal = "".
- progress-SLC/lib/updatetotals2.p — IF NOT AVAILABLE(Totals) THEN DO: CREATE Totals. ASSIGN (same fields) po-RetVal = "".

# Impacted Systems

Totals table, lib/updatetotals.p, lib/updatetotals2.p, serial add flow.

# Traceability

- [[BR-Totals-001-Shift-Scale-ProductCode-PdnDate-unique|BR-Totals-001]]
- [[../Data Models/0.Legacy Database Schema]] — Totals (Shift, Scale, ProductCode, PdnDate, LocalCount, LocalLabelWgt, LocalNetWgt, LocalTareWgt)

# Assertions

- Totals row is created only when the operation is Add and no row exists for (Shift, Scale, ProductCode, PdnDate).
- Initial LocalCount = 1; LocalLabelWgt, LocalNetWgt, LocalTareWgt come from the serial record.

# Related Information

