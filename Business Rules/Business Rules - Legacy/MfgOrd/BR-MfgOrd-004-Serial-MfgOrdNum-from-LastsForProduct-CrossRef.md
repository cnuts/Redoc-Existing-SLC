---
Risk-Level: P0
Business-Rule-Id: BR-MfgOrd-004
Deprecated: false
---

# Description

When creating a Serial record (createserial.p), Serial.MfgOrdNum and Serial.ProducedAtVRT are set from the CrossRef record with Application = "LastsForProduct" and ID = "MfgOrdNum". If that CrossRef exists and the new Serial's SerialNum does not start with "00", Serial.MfgOrdNum is set to INT64(CrossRef.Descr) and ProducedAtVRT is set to true; otherwise Serial.MfgOrdNum is 0 and ProducedAtVRT is false. Thus only "real" production serials (non–reprint/special SerialNum prefix) are linked to the selected manufacturing order.

# Source

- progress-SLC/lib/createserial.p: DEFINE BUFFER b-LastForMfgOrd FOR CrossRef; FIND FIRST b-LastForMfgOrd WHERE Application = "LastsForProduct" AND ID = "MfgOrdNum" NO-LOCK NO-ERROR; assign pb-Serial.MfgOrdNum = (IF AVAILABLE(b-LastForMfgOrd) AND SUBSTRING(pb-Serial.SerialNum,1,2) <> "00" THEN INT64(b-LastForMfgOrd.Descr) ELSE 0), pb-Serial.ProducedAtVRT = (IF SUBSTRING(pb-Serial.SerialNum,1,2) <> "00" THEN AVAILABLE(b-LastForMfgOrd) ELSE FALSE)

# Impacted Systems

Serial table (MfgOrdNum, ProducedAtVRT), CrossRef (LastsForProduct, MfgOrdNum), createserial.p, select-mfgord.w (sets the CrossRef).

# Traceability

- CrossRef.Descr holds the MfgOrdNum as a string; CreateSerial converts to INT64 for Serial.MfgOrdNum.
- SerialNum prefix "00" is used to exclude reprints or special cases from MfgOrd linkage and from ProducedAtVRT.

# Assertions

- Serial.MfgOrdNum is 0 when CrossRef LastsForProduct/MfgOrdNum is missing or when SerialNum starts with "00".
- ProducedAtVRT is true only when that CrossRef exists and SerialNum first two characters are not "00".

# Related Information

- [[BR-MfgOrd-001-ProdCode-and-Serial-relationship]]
- [[BR-MfgOrd-003-Select-MfgOrd-dialog-Open-ProdCode-date-and-LastsForProduct]]
