---
Risk-Level: P0
Business-Rule-Id: BR-Device-002
Deprecated: false
---

# Description

For input devices (scales/indicators), Device.Model identifies the indicator driver; the application loads indicators/<Model>.p (or .r). If Model = 'DEMO', no indicator program is loaded. If neither .p nor .r is found, "Invalid Indicator" / "There is no program available for this Wgt Indicator" is shown. For output devices, ComPort = 0 means network printer; output uses Device.NetworkID (e.g. Windows printer name) via Print-To-Network.p.

# Source

- [[../Data Models/Device/Device - legacy]] — §4 Input Devices, §5 Output Devices, SetIndicatorHandle, SendOutput.p
- weigh-setindicatorhandle.p; OpenOutputDevice

# Impacted Systems

Device table (Model, ComPort, NetworkID), indicators folder, SendOutput.p, Print-To-Network.p.

# Traceability

- Device.Model x(15); ComPort 0 = network; NetworkID x(15) for network printer name

# Assertions

- Input: Device.Model must match base name of existing indicator program in indicators/ or be DEMO.
- Output: ComPort = 0 → use NetworkID for network printer; else use COM port settings (Baud, Parity, DataBits, StopBits, etc.).
- If Device.Attached = FALSE, WriteToDevice returns without sending.

# Related Information

