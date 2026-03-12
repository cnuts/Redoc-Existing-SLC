---
Risk-Level: P0
Business-Rule-Id: BR-Locals-005
Deprecated: false
---

# Description

Access to Local Codes setup and product selection in s-localcodes.w is controlled by OperatorAppFunction permissions. If the operator does not have permission for "LocalCodes.Setup", the Setup button is hidden. If the operator does not have permission for "LocalCodes.Product", the Product button is hidden. Thus setup mode and the ability to assign products to local code slots are gated by these application function IDs.

# Source

- progress-SLC/sobjects/s-localcodes.w: IF NOT f-get-permission('LocalCodes.Setup') THEN btnSetup:hidden = true. IF NOT f-get-permission('LocalCodes.Product') THEN btnProduct:hidden = true

# Impacted Systems

s-localcodes.w, OperatorAppFunction (LocalCodes.Setup, LocalCodes.Product), f-get-permission (lib/services.p).

# Traceability

- LocalCodes.Setup controls visibility of the Setup button (run mode vs setup mode).
- LocalCodes.Product controls visibility of the Product button (assign product to local code).

# Assertions

- When permission is denied, the corresponding button is hidden; no other UI change is applied in this block.
- f-get-permission respects CFSUSER and SecurityOverride (see [[BR-Operator-005-f-get-permission-CFSUSER-and-SecurityOverride-bypass]]).

# Related Information

- [[BR-Locals-001-LocalCode-PageNbr-unique-ProductCode-range]]
- [[BR-Operator-001-Permission-gate]]
