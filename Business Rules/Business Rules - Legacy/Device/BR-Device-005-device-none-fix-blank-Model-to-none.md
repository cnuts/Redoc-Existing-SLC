---
Risk-Level: P0
Business-Rule-Id: BR-Device-005
Deprecated: false
---

# Description

The utility device-none-fix.p updates every Device row where the condition (in the source: "for each device where :" with no explicit predicate) applies: if Device.Model is blank (''), it assigns Device.Model = 'none'. After the update it displays DeviceID and Model in an alert-box. This normalizes blank Model values to the string 'none', which may be used elsewhere (e.g. indicator logic) to mean "no indicator program" or a sentinel value.

# Source

- progress-SLC/utilities/device-none-fix.p: FOR EACH Device WHERE (implied all): IF Device.Model = '' THEN Device.Model = 'none'. MESSAGE DeviceID Device.Model VIEW-AS ALERT-BOX.

# Impacted Systems

Device table (Model), utilities/device-none-fix.p.

# Traceability

- Run manually or as a one-off fix; not part of normal host or UI flow.
- Model x(15); blank may be invalid for logic that expects a value or 'DEMO'/'none'.

# Assertions

- Only Device.Model = '' is changed to 'none'; non-blank Model is unchanged.
- The MESSAGE runs for each Device row processed (per the source structure).

# Related Information

- [[BR-Device-002-Model-indicator-and-ComPort]]
