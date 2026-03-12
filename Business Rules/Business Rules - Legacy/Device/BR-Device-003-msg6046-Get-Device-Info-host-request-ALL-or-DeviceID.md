---
Risk-Level: P0
Business-Rule-Id: BR-Device-003
Deprecated: false
---

# Description

The host message handler msg6046.p (Get Device Information) responds to a request whose CommDtl.Body contains a device name (DeviceID) or "ALL". It finds Device where Device.DeviceID = production.CommDtl.Body; if a row is found or Body = "ALL", vFoundFile = yes, otherwise vFoundFile = no. If production.CommDtl is not available or vFoundFile is no, the procedure creates a response CommDtl (op-CommDtl) with Body = 'No CommDtl Provided or Device Not Found' and opReturnMsg set to that text, then returns (output TransType 6047). Otherwise it runs SendDeviceInfo: writes device data to temp/deviceinfo.txt (if Body <> "ALL" then DISPLAY the single Device row; if Body = "ALL" then FOR EACH Device NO-LOCK BREAK BY device.DeviceID DISPLAY each with separators), then reads that file and creates one op-CommDtl row per line with Body = the line text. Thus the host can request one device by name or all devices; a missing or invalid device name results in the "Device Not Found" response.

# Source

- progress-SLC/host/msg6046.p: FIND Device WHERE Device.DeviceID = CommDtl.Body; if avail Device OR Body = "ALL" then vFoundFile = yes else no. If not avail CommDtl or not vFoundFile then CREATE op-CommDtl, Body = 'No CommDtl Provided or Device Not Found', opReturnMsg = that, return. SendDeviceInfo: OUTPUT to temp/deviceinfo.txt; IF Body <> "ALL" FIND Device by Body, DISPLAY Device; ELSE FOR EACH device NO-LOCK BREAK BY device.DeviceID DISPLAY; INPUT from file, IMPORT unformatted vRecordText, CREATE op-CommDtl, Body = vRecordText. opTransType init 6046, return 6047.

# Impacted Systems

Device table, host/msg6046.p, production.CommDtl, temp/deviceinfo.txt, host/ici-pdn (caller).

# Traceability

- Request body = DeviceID or "ALL"; response is 6047 with device details or error message in Body.
- Output format is unformatted DISPLAY of Device fields, one record per line when "ALL".

# Assertions

- When Body is a specific DeviceID and no Device row exists, vFoundFile = no and the error response is returned.
- When Body = "ALL", no single-device lookup is required and all Device rows are written to the temp file and returned in CommDtl rows.

# Related Information

- [[BR-Device-001-DeviceID-unique-ProductProcess-FK]]
