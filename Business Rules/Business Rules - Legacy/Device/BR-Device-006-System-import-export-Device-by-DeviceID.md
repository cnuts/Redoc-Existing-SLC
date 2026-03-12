---
Risk-Level: P0
Business-Rule-Id: BR-Device-006
Deprecated: false
---

# Description

Device is included in the system export and import. Export writes the Device table to a file when "Device" is selected from the export table list. Import uses a temp-table TDevice like Device and matches existing rows by DeviceID: KeyPhrase is "Device.DeviceID = TDevice.DeviceID". The import file name is device.d. The counter vDevice tracks the number of Device records imported for the result message.

# Source

- progress-SLC/system/export.w: Device in export table list; WHEN 'Device' export with TableName Device
- progress-SLC/system/import.w: TDevice like Device; vDevice counter; WHEN 'Device' import from device.d, KeyPhrase Device.DeviceID = TDevice.DeviceID

# Impacted Systems

Device table, system/export.w, system/import.w, device.d.

# Traceability

- DeviceID is the primary key of Device; import uses it to update or insert.
- Dump name in slc.df is "device".

# Assertions

- Export dumps all Device rows (or per export selection) to the configured file.
- Import matches on DeviceID to update existing or create new Device rows.

# Related Information

- [[BR-Device-001-DeviceID-unique-ProductProcess-FK]]
