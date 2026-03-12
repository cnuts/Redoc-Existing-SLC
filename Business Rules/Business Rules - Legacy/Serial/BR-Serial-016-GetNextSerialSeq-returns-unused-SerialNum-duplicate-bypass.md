---
Risk-Level: P0
Business-Rule-Id: BR-Serial-016
Deprecated: false
---

# Description

Serial number generation in lib/createserial.p ensures that the returned SerialNum does not already exist in the Serial table. GetNextSerialSeq (and GetNextSampleSerialSeq) compute a candidate from PlantID(2) + ScaleID or AlternateScaleID(2) + JulianDate(4) + counter(4), or with UseSerialNum8: PlantID(2) + ScaleID(2) + 8-digit counter. They loop (v-SerialNumIsUsed = yes initially) incrementing UniqueNum and recomputing the candidate until FIND first pb-serial where SerialNum = poLastSerialCreated returns no record. When an existing serial is found, Logger("Dup ser:" + poLastSerialCreated + ":A") is run and UniqueNum is updated so the next candidate is tried. When more than one candidate was skipped (v-SerialsBypassed > 1), lib/send-6052-duplicate-serials.p is run with the bypass count and the final SerialNum so the event can be reported (e.g. email). UniqueNum is never left so that the 4-digit suffix is "0000" (intNextSeq modulo 10000 = 0 is replaced with 1).

# Source

- progress-SLC/lib/createserial.p — GetNextSerialSeq: do while v-SerialNumIsUsed = yes: v-NextUniqueNum = INT(f-get-session-parm("INT","UNIQUENUM")) + 1, intNextSeq = v-NextUniqueNum modulo lv-CounterLimit; if intNextSeq = 0 then intNextSeq = 1; build poLastSerialCreated; find first pb-serial where pb-serial.SerialNum = poLastSerialCreated no-error; if avail(pb-serial) then Logger and set UNIQUENUM; else v-SerialNumIsUsed = false. After loop: if v-SerialsBypassed > 1 then run lib/send-6052-duplicate-serials.p (input v-SerialsBypassed - 1, input poLastSerialCreated).

# Impacted Systems

Serial table, lib/createserial.p, GetNextSerialSeq, GetNextSampleSerialSeq, UniqueNum (session/site config), lib/send-6052-duplicate-serials.p.

# Traceability

- [[BR-Serial-001-Unique-SerialNum-AddOrDel|BR-Serial-001]] — uniqueness of (SerialNum, AddOrDel); this rule describes how SerialNum is chosen to avoid duplicates
- [[../Data Models/Serial/Serial - legacy]] — Serial ID format PlantID(2)+ScaleID(2)+JulianDate(4)+counter(4)

# Assertions

- The SerialNum returned by GetNextSerialSeq/GetNextSampleSerialSeq is not already present in the Serial table (for the same or "00"+substring form where applicable).
- When one or more candidates were skipped as duplicates, send-6052-duplicate-serials.p is invoked with the bypass count.
- The 4-digit counter portion (or 8-digit when UseSerialNum8) is never "0000".

# Related Information

