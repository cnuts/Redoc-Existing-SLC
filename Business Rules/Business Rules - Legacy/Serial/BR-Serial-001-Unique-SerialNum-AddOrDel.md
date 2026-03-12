---
Risk-Level: P0
Business-Rule-Id: BR-Serial-001
Deprecated: false
---

# Description

No two "A" (add) records may have the same SerialNum. SerialNum can only be created once. Only one "D" (delete) record is allowed per "A" record. AddOrDel may only contain "A" or "D"; if caps(ip-field-val) is not "A" and not "D", lib/val-serial.p returns error "Val.Serial.AddOrdDel.NotAorD".

# Source

- [[../Data Models/Serial/Serial - legacy]] — Business Rules for Serial (Creation Rules, State Rules)
- Database: Primary index UNIQUE (SerialNum, AddOrDel)
- progress-SLC/lib/val-serial.p — case "AddOrDel": if not( caps(ip-field-val) = "A" or caps(ip-field-val) = "D") then op-error = "Val.Serial.AddOrdDel.NotAorD"

# Impacted Systems

Serial table, ItemSerial table, lib/val-serial.p, createserial.p, cancellation and delete flows.

# Traceability

- [[../Data Models/0.Legacy Database Schema]] — Serial table, index Serial
- Effective production = Sum of [A records] − Count of [D records]

# Assertions

- Primary key (SerialNum, AddOrDel) is unique.
- For a given SerialNum, at most one row with AddOrDel = 'A'.
- For a given SerialNum, at most one row with AddOrDel = 'D' (cancellation).
- AddOrDel must be "A" or "D" (case-insensitive); when validation fails, op-error = "Val.Serial.AddOrdDel.NotAorD".
- SerialNum is often system-generated to ensure uniqueness.

# Related Information

