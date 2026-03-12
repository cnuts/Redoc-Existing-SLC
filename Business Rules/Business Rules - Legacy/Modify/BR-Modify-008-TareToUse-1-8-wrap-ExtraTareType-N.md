---
Risk-Level: P0
Business-Rule-Id: BR-Modify-008
Deprecated: false
---

# Description

In the Modify screen, TareToUse is treated as a value in the range 1–8. DisplayTare (s-modify.w) wraps values greater than 8 back to 1. When the product's ExtraTareType is 'N' (none), the tare selector shows 0 and Modify.TareToUse is not used for display; when ExtraTareType is 'W' or 'P', the display cycles through WgtTare1–8 or PercentTare1–8, and when gSkipZeroTares is true the procedure may try up to 8 times to find a non-zero tare value, wrapping 8→1 as needed.

# Source

- progress-SLC/sobjects/s-modify.w: vTareToUse init 1; BuildModifyFields assigns vTareToUse = (if tt-product.ExtraTareType = 'N' then 0 else modify.TareToUse); DisplayTare procedure: if pTareToUse > 8 then pTareToUse = 1; ExtraTareType 'n' shows 'ExtraTare=N'; 'W'/'P' cases use pTareToUse 1–8 and wrap pTareToUse > 8 → 1 when advancing; gSkipZeroTares and vTaresToTry = 8 for up to 8 attempts

# Impacted Systems

s-modify.w, Modify.TareToUse, Product (ExtraTareType, WgtTare1–8, PercentTare1–8), operator UI for tare selection.

# Traceability

- TareToUse 1–8: schema format allows wider range but UI and DisplayTare restrict to 1–8 with wrap.
- ExtraTareType 'N': vTareToUse = 0 and filTare shows 'ExtraTare=N'; saved Modify.TareToUse is not used for display in that case.

# Assertions

- When pTareToUse > 8, it is set to 1 before use in DisplayTare.
- When advancing (e.g. skip zero tares), after pTareToUse + 1, if pTareToUse > 8 then pTareToUse = 1.
- Product WgtTare1–8 and PercentTare1–8 are read by case pTareToUse 1–8.

# Related Information

- [[BR-Modify-003-Modify-screen-permissions]]
- [[BR-Product-004-ExtraTareType-and-ProductProcess]]
