---
Risk-Level: P0
Business-Rule-Id: BR-Device-001
Deprecated: false
---

# Description

Device.DeviceID is the primary key and must be unique. Every ProductProcess.Device and ProductProcess.AlternateDevice (when set) must reference a row in the Device table. Validation (validateprodprocess.p) returns an error if Device or AlternateDevice is specified but not found in Device.

# Source

- [[../Data Models/Device/Device - legacy]] — §2 Schema, §3 ProductProcess Link to Device
- [[../Data Models/Product-Process/Product-Process legacy]] — Validation Rules (ProductProcess-RI.p)
- print/validateprodprocess.p

# Impacted Systems

Device table, ProductProcess (Device, AlternateDevice), validateprodprocess.p, weigh.w (OpenDevice), SendOutput.p.

# Traceability

- Device index DeviceID unique ascending
- "Device: <DeviceID> | <ProcessSequence> not found" when Device missing

# Assertions

- DeviceID character x(20), unique primary key.
- ProductProcess.Device and AlternateDevice (if non-blank) must exist in Device.
- For input steps: Device must exist and Model must resolve to an indicator program (or DEMO).
- For output: if Device.Attached = FALSE, WriteToDevice does not send output.

# Related Information

