# Device Concept and Usage in the legacy SLC Application

This document describes the **Device** concept in full: the Device table schema, how devices are used for **scale/indicator input** and **label printer output**, how they are linked to **ProductProcess**, and how they relate to Serial, Label, Product, and other concepts.

---

## 1. Overview

A **Device** in the SLC is a hardware endpoint that the application talks to over serial (COM) or network. The same **Device** table is used for two distinct roles:

1. **Input devices (scales/indicators)**: Used in **input** ProductProcess steps. The **Device** record supplies **ComPort**, **Baud**, **Parity**, **DataBits**, **StopBits**, **FlowControl**, **DTREnable**, **InputBufferSize**, and **Model**. The **Model** (e.g. `GSE-450`, `DEMO`, `rice-lake-355`) determines which **indicator driver** (e.g. `indicators/<Model>.p`) is loaded to parse weight messages from the scale.
2. **Output devices (label printers)**: Used in **output** ProductProcess steps. The same table supplies COM settings (or **ComPort = 0** for network printers). **SendOutput.p** opens the port and sends the label stream (**WriteToDevice**) to the printer; for **ComPort = 0**, output goes via **Device.NetworkID** (e.g. Windows printer name) through **print/Print-To-Network.p**.

**ProductProcess.Device** (and optionally **ProductProcess.AlternateDevice**) stores the **DeviceID** (character) that references **Device.DeviceID**. Validation ensures that every **ProductProcess.Device** (and **AlternateDevice**, if set) exists in the **Device** table. The **Device** concept therefore ties the product process engine (weigh/print steps) to physical hardware.

---

## 2. Device Table Schema

**Source**: `slc.df`, table **Device**

| Field | Type | Format / Notes |
|-------|------|----------------|
| **DeviceID** | character | x(20), initial "", **primary unique index** |
| **ComPort** | integer | >>9, initial 0. **0 = network printer**; non-zero = COM port number |
| **NetworkID** | character | x(15). For network printers (ComPort=0), Windows printer name or network path used by Print-To-Network.p |
| **IPAddress** | character | x(15) |
| **Parity** | character | x(8). Serial parity (e.g. N, E, O) |
| **Baud** | integer | Serial baud rate |
| **DataBits** | integer | Serial data bits (e.g. 7, 8) |
| **StopBits** | integer | Serial stop bits |
| **TimeoutMS** | integer | Timeout in milliseconds |
| **Model** | character | x(15). **For input devices**: name of indicator driver, e.g. `GSE-450`, `DEMO`, `rice-lake-355`. Used to load `indicators/<Model>.p` or `.r`. |
| **FlowControl** | logical | yes/no. Serial flow control (handshaking) |
| **InputBufferSize** | integer | For serial input (scale) buffer |
| **OutputBufferSize** | integer | For serial output (printer) buffer |
| **DTREnable** | logical | DTR enable for serial |
| **Descr** | character | x(25). Description |
| **GenericPrinter** | logical | "Generic is like an HP line prtr, vs something like an I-class label printer" |
| **Attached** | logical | yes/no. **If FALSE**, **WriteToDevice** in SendOutput.p does not open or write to the device (output is skipped). For input, device must exist and Model must resolve to an indicator program. |

**Index**: **DeviceID** (unique, ascending).

---

## 3. ProductProcess Link to Device

**Source**: `slc.df`, table **ProductProcess**

- **Device** (character, x(15)): The **DeviceID** of the device used for this process step. Required for steps that use hardware (scale or printer).
- **AlternateDevice** (character, x(15)): Optional alternate **DeviceID**. If non-blank, it is validated to exist in **Device** (see **validateprodprocess.p**).

**Validation** (`print/validateprodprocess.p`): When validating a product process configuration, the program finds **Device** where **Device.DeviceID = TT-ProductProcess.Device**. If no row exists, it returns an error (e.g. "Device: &lt;DeviceID&gt; | &lt;ProcessSequence&gt; not found"). The same check is done for **TT-ProductProcess.AlternateDevice** when it is set. So every **ProductProcess.Device** and **ProductProcess.AlternateDevice** must reference a row in **Device**.

**Usage in flow**: For an **input** step (e.g. get weight), **weigh.w** calls **OpenDevice(tt-ProductProcess.Device, ipInputProcess)** and uses that device for the scale COM port and indicator driver. For an **output** step (e.g. print label), **case-print.p** (and siblings) pass **b-tt-ProductProcess.Device** as **piDeviceID** to **PrintLabel** and **WriteToDevice**; **iopLastSerialPrinterID** is set to **b-tt-ProductProcess.Device** so the next cancel or reprint can use the same printer.

---

## 4. Input Devices (Scale / Indicator)

### 4.1 Role

Input steps in the product process (e.g. "get weight") need a **scale (indicator)**. The **ProductProcess.Device** for that step points to a **Device** row whose **Model** identifies the indicator type. The application:

1. Opens the COM port using **Device** settings (ComPort, Baud, Parity, DataBits, StopBits, FlowControl, DTREnable, InputBufferSize).
2. Loads the **indicator driver** from **indicators/&lt;Device.Model&gt;.p** (or .r). Examples: **GSE-450**, **rice-lake-355**, **toledo-560**, **DEMO** (DEMO skips loading a real driver).
3. Uses that driver (**gh_ParseIndicator**) to parse incoming data and return weight (and optionally other values).

### 4.2 OpenDevice (weigh.w)

**Procedure OpenDevice** (input **ipDeviceID**, **ipInputProcess**):

- If **ipDeviceID** = **'ProductScanner1'**, returns without opening (scanner has its own logic).
- If **f-ProductGetsComInput()** is false (e.g. demo mode), returns.
- If **ipDeviceID** equals **vLastInputDevice** (same device as last time), may skip re-opening.
- **FIND Device WHERE Device.DeviceID = ipDeviceID**. If not available, shows message "MissingDevice" and quits.
- Calls **SetIndicatorHandle** in **gh_Weigh-SetIndicatorHandle** with **buffer Device**. That routine loads **indicators/&lt;Device.Model&gt;.p** (or .r) and sets **gh_ParseIndicator**.
- The weigh window then uses the COM control (e.g. **chCtrlFrame:MSComm** for input) and **gh_ParseIndicator** to read and parse scale data.

**vLastInputDevice**: Tracks the current input device so the same port is not reopened unnecessarily. **CloseIndicator** resets **vLastInputDevice** to null so the next step can open the correct device.

### 4.3 SetIndicatorHandle (weigh-setindicatorhandle.p)

- **Input**: **buffer Device** (the Device row for the current step).
- If **Device.Model** = **'DEMO'**, no indicator program is loaded (**gh_ParseIndicator** remains unset or is cleared).
- Otherwise: **vIndicatorPgm** = **'indicators/' + Device.Model + '.p'** (or .r). **SEARCH(vIndicatorPgm)** resolves the path (including procedure library paths). If neither .p nor .r is found, shows "Invalid Indicator" / "There is no program available for this Wgt Indicator".
- **RUN VALUE(vIndicatorPgm) PERSISTENT SET gh_ParseIndicator**. So **Device.Model** must match the base name of an existing indicator program in the **indicators/** folder.

---

## 5. Output Devices (Label Printer)

### 5.1 Role

Output steps (e.g. print case label, print cancel label) send data to a **printer**. The **ProductProcess.Device** for that step is the **DeviceID** of the label printer. The application:

1. Resolves the COM handle for the printer (e.g. **ctlOutputDevice** in weigh.w, or **ctlLabelPrinter** in s-setupprinter.w — often an **MSComm** control).
2. When printing, **PrintLabel** (lib/printlabel.p) is called with **piDeviceID** = **b-tt-ProductProcess.Device** and **pih_Printer** = that COM handle.
3. **WriteToDevice** in **gh_SendOutput** (SendOutput.p) looks up **Device** by **piDeviceID**. If **Device** is not available or **Device.Attached** is FALSE, it returns without sending. Otherwise it opens the device (if needed) and sends the raw label (**piMessage**) to the printer.

### 5.2 SendOutput.p

**WriteToDevice** (input **pih_Device** COM-HANDLE, **piMessage** RAW, **piPlainText** char, **piDeviceID** char):

- **FIND b-Device WHERE b-Device.DeviceID = piDeviceID**. If not available or **b-Device.Attached = FALSE**, returns (no output).
- Calls **OpenOutputDevice(pih_Device, piDeviceID, output vUseNetworkPrinter)**.

**OpenOutputDevice**:

- **FIND Device WHERE Device.DeviceID = piDeviceID**. If not available, message and return.
- If **Device.ComPort = 0**: sets **poUseNetworkPrinter = TRUE** and returns (no COM open).
- Otherwise: builds **vPrinterSettings** from **Device.Baud**, **Parity**, **DataBits**, **StopBits**. Sets **pih_Device:CommPort**, **OutBufferSize**, **DTREnable**, **Settings**, **Handshaking** from **Device**, then **pih_Device:PortOpen = TRUE**. In **gInMotionGetProduct** mode, if the port is already open, it may skip reopening.

Back in **WriteToDevice**:

- If **vUseNetworkPrinter**: runs **print/Print-To-Network.p** with **Device.NetworkID** (printer name) and **piMessage**. So for network printers, **Device.NetworkID** is the key; **ComPort = 0** is the convention.
- Else: sends **piMessage** (or **piPlainText**) to **pih_Device:output** (the COM port).

**CloseOutputDevice** (input **pih_Device**): sets **pih_Device:PortOpen = FALSE**.

### 5.3 Setup Printer and Device Selection

**sobjects/s-setupprinter.w**:

- **ctlLabelPrinter**: COM handle (e.g. **chCtrlFrame:MSComm**) for the label printer.
- **cbDevices**: Combo box populated with all **Device.DeviceID** values (**ip-InitDevices**: **FOR EACH Device**, **cbDevices:add-last(Device.DeviceID)**). Default screen-value is **'CasePrinter'** (no-error).
- When testing setup, **WriteToDevice** is called with **ctlLabelPrinter** and **cbDevices:SCREEN-VALUE** (the selected DeviceID). So the user selects which **Device** record (and thus which COM/network settings) the label printer uses.

**weigh.w**: **ctlOutputDevice** is the output COM handle (e.g. **chCtrlFrame-2:MSComm**). When a case label is printed, **b-tt-ProductProcess.Device** is passed as **piDeviceID** to **PrintLabel** and thence to **WriteToDevice**. So the **product process** step, not the setup window, decides which Device (printer) is used for that step; the same machine may have multiple Device records for different printers.

---

## 6. Default and Reprint Device

- **reprint-case-lbl.w**: **vDefaultDevice** is initialized to **'CasePrinter1'**. When reprinting, **PrintLabel** is called with that device ID (**input vDefaultDevice**) so reprints use a known default printer unless the UI allows changing it.
- So **DeviceID** values like **CasePrinter1**, **CasePrinter** are conventional names for the default label printer(s).

---

## 7. Other Device Usages

- **divert.p**: Sends a signal to divert the object to the **ProductProcess.Device.ComPort** (i.e. the device’s COM port).
- **op-send-stop-to-indicator.p**: Sends a stop message to the indicator associated with **ProductProcess.Device** (ComPort).
- **itemlabel.p**: Accepts **ipDeviceID** and passes it to the print/output logic for item labels.
- **palletlabel.p**: **piDeviceID** is passed when printing pallet labels (e.g. to **WriteToDevice**).
- **s-rejectionsetup.w**: Uses **WriteToDevice** and **CloseOutputDevice** with **ctlDevice** for rejection/divert hardware.

So **Device** is used wherever the app needs to talk to a specific COM or network printer/indicator/divert device; the **DeviceID** is the stable key.

---

## 8. Relationships to Other Concepts

| Concept | Relationship to Device |
|--------|------------------------|
| **ProductProcess** | **ProductProcess.Device** and **AlternateDevice** store **Device.DeviceID**. Each input or output step that uses hardware references one (or two) Device(s). Validation ensures the Device row exists. |
| **Label** | The label stream is sent to the output device via **WriteToDevice(handle, raw, "", DeviceID)**. **ProductProcess.Device** for the print step is the **piDeviceID** passed to **PrintLabel** and **WriteToDevice**. **Device.Attached** and **Device.ComPort** (0 = network) control whether and how output is sent. |
| **Serial** | Serial creation and printing are driven by the product process; the step that prints uses **ProductProcess.Device**. **iopLastSerialPrinterID** is set to that Device so cancel/reprint use the same printer. |
| **Indicator / Scale** | **Device.Model** selects the indicator driver (**indicators/&lt;Model&gt;.p**). **Device** COM settings configure the serial port used to read weight data. |
| **weigh.w** | Opens input device via **OpenDevice(DeviceID)** and uses **gh_ParseIndicator** (from **Device.Model**). Uses **ctlOutputDevice** and **ProductProcess.Device** when calling case-print/PrintLabel. |
| **Config / Site** | **gPrinterAttached** (site/config) can prevent any physical printer output; **WriteToDevice** may still be called but device open/write logic may depend on **Device.Attached** and handle validity. |

---

## 9. Business Rules Summary

- **DeviceID** is unique. Every **ProductProcess.Device** and **ProductProcess.AlternateDevice** must exist as **Device.DeviceID** (enforced in **validateprodprocess.p**).
- **Input device**: **Device** must exist; **Device.Model** must match an existing **indicators/&lt;Model&gt;.p** or **.r** (or **DEMO**). COM settings from **Device** are used to open the scale port.
- **Output device**: **Device** must exist; **Device.Attached** must be TRUE for **WriteToDevice** to send. **ComPort = 0** means network printer; **Device.NetworkID** is then used. Otherwise COM settings from **Device** are applied to the COM handle before opening.
- **Reprint**: Uses a default **DeviceID** (e.g. **CasePrinter1**) unless overridden by UI.

---

## 10. UI and Maintenance

- **sobjects/v-device.w**: SmartViewer for **Device** table. Displays/edits **DeviceID**, **Descr**, **ComPort**, **Baud**, **Parity**, **DataBits**, **StopBits**, **FlowControl**, **DTREnable**, **Model**, **IPAddress**, **NetworkID**, **InputBufferSize**, **OutputBufferSize**, **TimeoutMS**, **GenericPrinter**, **Attached**. **Device.Model** has a delimiter list (e.g. for selecting from known indicator names). Add/Copy/Delete Device with validation (e.g. "Please set Device ID and other values").
- **system/import.w**, **system/export.w**: Device table can be included in import/export (dump name "Device").
- **s-setupprinter.w**: **cbDevices** lists all DeviceIDs; user selects which device the label printer COM handle corresponds to for setup/test.

---

## 11. Redevelopment Notes

- Implement **Device** table (or equivalent) with **DeviceID** as primary key and all COM/network and **Attached**/ **Model** fields.
- **ProductProcess** must store **Device** (and optionally **AlternateDevice**) as foreign key to **Device.DeviceID**; validate on save.
- **Input path**: Resolve **Device** by step’s **Device**; open serial using **Device** COM fields; load driver from **indicators/&lt;Device.Model&gt;** (or equivalent); parse weight from stream.
- **Output path**: Resolve **Device** by step’s **Device**; if **Attached** and handle valid, open (COM from **Device** or **ComPort=0** for network); send label stream to COM or to **Device.NetworkID** (network print).
- **Attached** and **gPrinterAttached** (or equivalent) allow disabling physical output without changing process configuration.
- **OpenOutputDevice** / **CloseOutputDevice** / **WriteToDevice** semantics (including “already open” skip for in-motion) should be preserved for parity.
