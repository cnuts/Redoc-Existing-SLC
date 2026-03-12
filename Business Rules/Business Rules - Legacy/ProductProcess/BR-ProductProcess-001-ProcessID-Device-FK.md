---
Risk-Level: P0
Business-Rule-Id: BR-ProductProcess-001
Deprecated: false
---

# Description

ProcessID must exist in ValidProcess table; if missing, error "Invalid ProcessID". Device (if specified) must exist in Device table; if missing, error "Device not found". AlternateDevice (if specified) must exist in Device table; if missing, warning or skip. If InputProcess = YES, Routine must be defined; if missing, error "Routine required for input". ProcessSequence must be unique per ProductCode; if duplicate, error "Duplicate ProcessSequence".

# Source

- [[../Data Models/Product-Process/Product-Process legacy]] — Validation Rules (ProductProcess-RI.p), Data Types and Constraints
- validateprodprocess.p

# Impacted Systems

ProductProcess, ValidProcess, Device, validateprodprocess.p, weigh-create-tt-prodprocess.p.

# Traceability

- Primary key (ProductCode, ProcessSequence); ProcessID FK to ValidProcess; Device, AlternateDevice FK to Device

# Assertions

- ProcessID FK to ValidProcess.
- Device and AlternateDevice (when set) FK to Device.
- InputProcess = YES implies Routine is defined.
- ProcessSequence unique per ProductCode.

# Related Information

