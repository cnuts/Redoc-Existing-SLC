---
Risk-Level: P0
Business-Rule-Id: BR-Modify-005
Deprecated: false
---

# Description

When producing to a work order (gProduceToWO), the UpdateFromGoal procedure in weigh-create-modify.p copies selected Goals fields into b-tt-Modify only when the goal value is not unknown (?). No other Modify or Pallet fields (e.g. PalletNum, PalletTracking) are set from Goals in this procedure.

# Source

- progress-SLC/print/weigh-create-modify.p: procedure UpdateFromGoal (buffer Goals, buffer Modify, buffer b-tt-modify); 400-Update transaction with IF goals.* <> ? THEN b-tt-Modify.* = goals.*

# Impacted Systems

tt-Modify (b-tt-modify), Goals table, weigh-create-modify.p (invoked from Create-Modify when gProduceToWO).

# Traceability

- Goals.CasesPerPallet → b-tt-Modify.CasesPerPallet
- Goals.CustomerName → b-tt-Modify.CustName
- Goals.OrdNum → b-tt-Modify.OrdNum
- Goals.OrdRefLine → b-tt-Modify.PdnOrdLineNum
- Goals.PackDateOffset → b-tt-Modify.PackDateOffset
- Goals.UnitPrice → b-tt-Modify.Price
- Goals.VarOffset → b-tt-Modify.VarOffset

# Assertions

- Each assignment is conditional: IF goals.<field> <> ? THEN b-tt-Modify.<field> = goals.<field>.
- Modify table buffer is not updated in UpdateFromGoal; only b-tt-modify is updated.
- UpdateFromGoal is run only when gProduceToWO is true, after Create-Modify has built tt-Modify from Modify.

# Related Information

- [[BR-Modify-002-GlobalKillDate-and-ZuluCapture]]
- [[BR-Modify-004-Create-Modify-defaults-and-tt-Modify-refresh]]
- [[BR-Goals-005-Host-download-and-Counter-Goal]]
