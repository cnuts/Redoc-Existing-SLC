---
Risk-Level: P0
Business-Rule-Id: BR-Item-007
Deprecated: false
---

# Description

When CreateSerial (createitemserial.p) successfully creates an ItemSerial record for an add (chrAddDel = 'a'), it sets ItemSerial.GoalID to Goals.GoalID when a Goals row is available (gGoalsRowid), otherwise to ?. After a successful transaction, gLastSerialNum is set to '' and gLastItemSerialNum is set to the new ItemSerial.SerialNum (pb-ItemSerial.SerialNum), so the case serial is cleared and the last item serial is updated for subsequent logic (e.g. cancel cannot target the case after an item serial is produced). When the host type is not Standalone, CurrentCountUp is incremented in session (f-set-session-parm) so the count reflects the new item serial.

# Source

- progress-SLC/lib/createitemserial.p: FIND Goals by gGoalsRowid; CREATE pb-ItemSerial; ASSIGN GoalID = (if avail Goals then Goals.GoalID else ?), ...; on success ASSIGN gLastSerialNum = '', gLastItemSerialNum = pb-ItemSerial.SerialNum; IF NOT v-Error AND gHostType <> Hosttype_Standalone THEN vCurCountUp = CurrentCountUp + 1, f-set-session-parm("CurrentCountUp", string(vCurCountUp,"99999"))
- Comment: "when create an item serial, clear gLastSerialNum, because can no longer cancel a case serial after an item serial is produced"

# Impacted Systems

ItemSerial, Goals, createitemserial.p, gLastSerialNum, gLastItemSerialNum, CurrentCountUp session parm, gHostType.

# Traceability

- GoalID links the item serial to the work order goal when producing to WO.
- Clearing gLastSerialNum prevents cancel-case from being used after an item has been printed.

# Assertions

- gLastItemSerialNum is updated only when po-Ok is true (no validation error).
- CurrentCountUp is not incremented when gHostType is Hosttype_Standalone.

# Related Information

- [[BR-Item-002-ItemSerial-CaseSerialNum-GoalID]]
