---
Risk-Level: P0
Business-Rule-Id: BR-Serial-015
Deprecated: false
---

# Description

When a Serial record is created with the reprint flag set (pb-serial.reprinted = true), Totals and Goals are not updated. lib/createserial.p runs lib/updatetotals2.p only when "v-Error EQ FALSE and not pb-serial.reprinted". So reprinted serials do not affect LocalCount, LocalLabelWgt, LocalNetWgt, LocalTareWgt, or Goal counts/weights.

# Source

- progress-SLC/lib/createserial.p — Procedure CreateSerial: IF v-Error EQ FALSE and not pb-serial.reprinted THEN RUN lib/updatetotals2.p(INPUT pb-Serial.SerialNum, INPUT chrAddDel, OUTPUT v-RetVal, BUFFER pb-Serial). Comment 12/12/02: "don't add to totals when serial.reprinted".

# Impacted Systems

Serial table (reprinted field), lib/createserial.p, lib/updatetotals2.p, Totals table, Goals table.

# Traceability

- [[BR-Totals-003-Create-Totals-on-first-add-when-no-row-exists|BR-Totals-003]]
- [[BR-Totals-001-Shift-Scale-ProductCode-PdnDate-unique|BR-Totals-001]]
- [[../Data Models/Serial/Serial - legacy]]

# Assertions

- When serial.reprinted is true, UpdateTotals (updatetotals2.p) is not called for that serial.
- Production counts and goal progress are not incremented for reprinted labels.

# Related Information

