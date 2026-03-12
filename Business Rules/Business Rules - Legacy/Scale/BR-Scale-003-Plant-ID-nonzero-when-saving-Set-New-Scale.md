---
Risk-Level: P0
Business-Rule-Id: BR-Scale-003
Deprecated: false
---

# Description

When saving configuration from the "Set New Scale" (Set Up New Scale) window, Plant ID must not be zero. If fiPlantID = 0, the application displays "Plant ID: 0 is not valid. Update not performed" and returns without updating site config (PlantID, ScaleID, UniqueNum, PDN, HostType). This prevents persisting an invalid plant for the scale.

# Source

- progress-SLC/sobjects/w-set-new-scale.w — procedure ip-Save: IF fiPlantID = 0 THEN DO: MESSAGE "Plant ID: " fiPlantID " is not valid." SKIP "Update not performed" SKIP VIEW-AS ALERT-BOX INFO BUTTONS OK. RETURN. END.

# Impacted Systems

sobjects/w-set-new-scale.w (btnSave → ip-Save), SiteConfigOption (PlantID), gPlantID.

# Traceability

- [[BR-Scale-001-Scale-ID-unique-per-plant]] — scale is identified per plant; plant must be valid
- Set New Scale UI: fiPlantID, fiScaleID, fiUniqueNum, PDN host/service, HostType

# Assertions

- On Save in Set New Scale, if Plant ID is 0, the update is aborted and the user is informed.
- No site config or globals (gPlantID, gScaleID, etc.) are updated when Plant ID is 0.

# Related Information

