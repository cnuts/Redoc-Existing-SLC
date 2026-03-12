---
Risk-Level: P0
Business-Rule-Id: BR-ProductProcess-004
Deprecated: false
---

# Description

When validating ProductProcess (print/validateprodprocess.p), Device must exist in the Device table. FIND first Device where Device.DeviceID = TT-ProductProcess.Device no-lock no-error; if not avail Device, run dialogs/d-tsmsgbox.w with message "Device: [DeviceID] | [ProcessSequence]" not found (PrintLabels.EMsg.FileMissing) and set poOK = no. If TT-ProductProcess.AlternateDevice is not ? and not "", then AlternateDevice must also exist in the Device table; if not avail Device for AlternateDevice, show same style message for "Alternate Device: [AlternateDevice] | [ProcessSequence]" and set poOK = no.

# Source

- progress-SLC/print/validateprodprocess.p — Procedure ValidateDevices: FIND Device where Device.DeviceID = TT-ProductProcess.Device; if not avail then message and poOK = no; same for AlternateDevice when set

# Impacted Systems

ProductProcess, Device table, print/validateprodprocess.p, s-printlbl.w (runs validation at start of each new product).

# Traceability

- [[../Drafts/ProductProcess/BR-ProductProcess-001-ProcessID-Device-FK]]
- [[../Data Models/Device/Device - legacy]]

# Assertions

- TT-ProductProcess.Device must reference an existing Device.DeviceID.
- When AlternateDevice is non-blank, TT-ProductProcess.AlternateDevice must reference an existing Device.DeviceID.
- When validation fails, user is shown Product Process Error for Product and FileMissing message; poOK = no.

# Related Information

