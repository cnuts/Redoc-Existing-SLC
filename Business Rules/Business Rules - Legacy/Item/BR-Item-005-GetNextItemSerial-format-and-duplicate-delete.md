---
Risk-Level: P0
Business-Rule-Id: BR-Item-005
Deprecated: false
---

# Description

GetNextItemSerial (createitemserial.p) generates the next item serial number and returns it in poLastSerialCreated. The format is: string(gPlantID,'99') + string(gScaleID,'99') + chrJulianDate (from f-LocalDateToJulian of Serial-Date site value) + '00' + string(intNextSeq,'9999999999'). ItemSequence is read from session, incremented by 1, and wrapped to 1 if it goes below 1 or above 1999999999; the new value is saved back to session. If an ItemSerial row already exists with that SerialNum and AddOrDel = 'A', it is deleted (duplicate) and the same number is used. Thus the procedure both allocates the next sequence and removes any prior duplicate add record that would conflict.

# Source

- progress-SLC/lib/createitemserial.p: GetNextItemSerial; vToday from Serial-Date site value; chrJulianDate = f-LocalDateToJulian(vToday); intNextSeq = ItemSequence + 1, wrap to 1 if GT 1999999999 or LT 1; f-set-session-parm ItemSequence; poLastSerialCreated = PlantID(99) + ScaleID(99) + chrJulianDate + '00' + string(intNextSeq,'9999999999'); FIND ItemSerial by SerialNum and AddOrDel 'A'; IF AVAIL DELETE pb-ItemSerial, RUN Logger("delete dup Item ser:" + ...)

# Impacted Systems

ItemSerial table, createitemserial.p, session parm ItemSequence, Serial-Date site value, gPlantID, gScaleID.

# Traceability

- The '00' in the middle distinguishes item serials from case serials in the numbering scheme.
- Duplicate delete ensures the returned number is available for a new Add record.

# Assertions

- ItemSequence is stored and updated via f-get-session-parm / f-set-session-parm.
- Only an existing 'A' (add) record with the same SerialNum is deleted; cancel (D) records are not removed by this procedure.

# Related Information

- [[BR-Item-002-ItemSerial-CaseSerialNum-GoalID]]
