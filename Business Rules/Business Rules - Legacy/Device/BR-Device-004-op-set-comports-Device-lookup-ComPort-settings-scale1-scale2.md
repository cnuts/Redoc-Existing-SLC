---
Risk-Level: P0
Business-Rule-Id: BR-Device-004
Deprecated: false
---

# Description

In op-set-comports.w, when resolving COM port and serial settings for the two scales, the procedure finds Device by DeviceID matching b-tt-productprocess.Device (Scale 1) and b-tt-productprocess.AlternateDevice (Scale 2). For each scale, if a Device row is available, it assigns the scale's ComPort, Settings (Baud, Parity, DataBits, StopBits as comma-separated string), and FlowControl (as 1 or 0) from the Device fields. If Device is not found for Scale 1, op-Msg is set to 'Scale 1 Device not found'. If Device is not found for Scale 2, the message 'Scale 2 Device not found' is appended to op-Msg. Thus ProductProcess.Device and ProductProcess.AlternateDevice must reference existing Device rows for the COM/serial settings to be applied; otherwise the user receives a "Device not found" message for that scale.

# Source

- progress-SLC/print/op-set-comports.w: FIND FIRST Device WHERE Device.DeviceID = b-tt-productprocess.Device NO-LOCK NO-ERROR; IF AVAIL Device then vScale1ComPort = Device.ComPort, vScale1Device = Device.DeviceID, vScale1Settings = TRIM(STRING(Device.Baud)) + "," + Device.Parity + "," + STRING(INT(Device.DataBits)) + "," + STRING(INT(Device.StopBits)), vScale1Handshaking = (if Device.FlowControl then 1 else 0); ELSE op-Msg = 'Scale 1 Device not found'. Same for AlternateDevice → vScale2ComPort, vScale2Device, vScale2Settings, vScale2Handshaking; else append 'Scale 2 Device not found'.

# Impacted Systems

Device table (ComPort, Baud, Parity, DataBits, StopBits, FlowControl), tt-ProductProcess (Device, AlternateDevice), op-set-comports.w, OpenDevices/CloseDevices.

# Traceability

- Scale 1 uses ProductProcess.Device; Scale 2 uses ProductProcess.AlternateDevice. Both must exist in Device for COM settings to be applied.
- Comment in file: Device field corresponds to Scale 1, AlternateDevice to Scale 2; WriteToDevice/sendoutput.p opens comport based on device settings.

# Assertions

- If either Device or AlternateDevice is not found, the corresponding scale's settings are not set and the error message is recorded in op-Msg.
- Settings string format: Baud,Parity,DataBits,StopBits (e.g. from Device.Baud, Device.Parity, etc.).

# Related Information

- [[BR-Device-001-DeviceID-unique-ProductProcess-FK]]
- [[BR-Device-002-Model-indicator-and-ComPort]]
