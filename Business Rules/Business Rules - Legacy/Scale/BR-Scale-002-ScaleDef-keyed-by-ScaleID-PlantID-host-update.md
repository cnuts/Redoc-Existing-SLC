---
Risk-Level: P0
Business-Rule-Id: BR-Scale-002
Deprecated: false
---

# Description

The ScaleDef table (host/production database) is identified by ScaleID and PlantID. host/updscaledef.p finds the ScaleDef row where ScaleDef.ScaleID = gScaleid and ScaleDef.PlantID = gPlant (exclusive-lock). If the row exists, it updates CountUp (from gCurrentCountUp), MsgDate (today), MsgTime (time formatted HH:MM, colons removed), Operator (gOperator), ProdCode (gCurrentProduct), and Shift (gShift). The procedure is run from the host (e.g. progress-host.w) per host interval; it does not create a ScaleDef row—it only updates an existing one. So each scale at a plant must have a pre-existing ScaleDef row for the host to update status.

# Source

- progress-SLC/host/updscaledef.p — procedure UpdScaleDef: do trans: find ScaleDef where ScaleDef.ScaleID = gScaleid and ScaleDef.PlantID = gPlant exclusive-lock no-error. if avail Scaledef then assign ScaleDef.CountUp = gCurrentCountUp, ScaleDef.MsgDate = today, ScaleDef.MsgTime = replace(string(time,"HH:MM"),":",""), ScaleDef.Operator = gOperator, ScaleDef.ProdCode = gCurrentProduct, ScaleDef.Shift = gShift.

# Impacted Systems

ScaleDef table (host DB), host/updscaledef.p, progress-host.w (ocx.tick), gScaleid, gPlant, gCurrentCountUp, gOperator, gCurrentProduct, gShift.

# Traceability

- [[BR-Scale-001-Scale-ID-unique-per-plant]] — ScaleID unique per plant; ScaleDef is the host-side record for that scale/plant
- Renamed from updhostinfo.p; run once per host interval

# Assertions

- ScaleDef is looked up by (ScaleID, PlantID).
- When found, ScaleDef is updated with current count-up and session values; when not found, no row is created.
- ScaleDef rows must exist for each scale/plant before the host can update them.

# Related Information

