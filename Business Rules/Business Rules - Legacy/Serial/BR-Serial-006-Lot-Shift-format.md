---
Risk-Level: P0
Business-Rule-Id: BR-Serial-006
Deprecated: false
---

# Description

Lot must be alphanumeric and maximum 10 characters. Shift must be in the range 01–99 (typical usage 1, 2, 3). ScaleNumber must be in the range 01–99. SerialNum must be 12 characters in length. Enforced in lib/val-serial.p: Shift out of range returns "Val.Serial.Shift.OutOfLimit"; ScaleNumber out of range returns "Val.Serial.ScaleNumber.OutOfLimit"; SerialNum length < 12 returns "Val.Serial.SerialNum.ShortLen".

# Source

- [[../Data Models/Serial/Serial - legacy]] — Data Integrity Rules, Data Validation Checklist
- [[../Data Models/0.Legacy Database Schema]] — Serial (Lot, Shift, SerialNum)
- progress-SLC/lib/val-serial.p — case "Shift", "ScaleNumber", "SerialNum" (range/length checks)

# Impacted Systems

Serial table, ItemSerial table, lib/val-serial.p, Totals (Shift, Scale in key), createserial.p, validation and reporting.

# Traceability

- Serial schema: Lot X(10), Shift 99, SerialNum X(12)
- val-serial.p: Shift > 0 and <= 99; ScaleNumber 01–99 (intended; code has logic bug "<= 0 and > 99"); SerialNum length >= 12

# Assertions

- Lot: alphanumeric, max 10 chars.
- Shift must be > 0 and <= 99 (range 01–99); when validation fails, op-error = "Val.Serial.Shift.OutOfLimit".
- ScaleNumber must be in range 01–99; when validation fails, op-error = "Val.Serial.ScaleNumber.OutOfLimit".
- SerialNum length of trimmed value must be >= 12; when validation fails, op-error = "Val.Serial.SerialNum.ShortLen".

# Related Information

