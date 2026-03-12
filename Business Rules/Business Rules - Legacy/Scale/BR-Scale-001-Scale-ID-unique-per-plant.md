---
Risk-Level: P0
Business-Rule-Id: BR-Scale-001
Deprecated: false
---

# Description

Scale ID numbers must be unique per plant. No two SLC devices (scales) at the same plant may share the same scale ID. The host finds ScaleDef by (ScaleID, PlantID); Serial.ScaleNumber and Totals.Scale store the scale at which the case was produced and are set from the session scale (gScaleID) at serial creation.

# Source

- [[../Business Rules/Scale IDs are unique per plant]]
- progress-SLC/host/updscaledef.p — FIND ScaleDef WHERE ScaleDef.ScaleID = gScaleid AND ScaleDef.PlantID = gPlant; one row per scale per plant
- progress-SLC/lib/createserial.p — pb-serial.ScaleNumber = gScaleID
- [[../Data Models/0.Legacy Database Schema]] — Serial.ScaleNumber, Totals.Scale (format 99)

# Impacted Systems

ScaleDef (host DB), Serial.ScaleNumber, Totals.Scale, lib/createserial.p, host/updscaledef.p, sobjects/w-set-new-scale.w (fiScaleID, fiPlantID), production and reporting.

# Traceability

- [[../Data Models/0.Legacy Database Schema]] (Serial.ScaleNumber, Totals.Scale)
- Scale number range 01–99 enforced in Serial context: [[BR-Serial-006-Lot-Shift-format]]

# Assertions

- No other SLC devices will share a scale ID value within the same plant.
- ScaleDef is keyed by ScaleID and PlantID; at most one ScaleDef row per (ScaleID, PlantID).

# Related Information

