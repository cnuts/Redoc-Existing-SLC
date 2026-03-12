---
Risk-Level: P0
Business-Rule-Id: BR-Modify-004
Deprecated: false
---

# Description

The Create-Modify procedure (weigh-create-modify.p) ensures a single tt-Modify row for the current product: it deletes all existing tt-Modify rows, finds or creates the Modify record by gTTProductCode, applies GlobalKillDate and ZuluDept per site config, buffer-copies Modify to b-tt-modify, and when gProduceToWO runs UpdateFromGoal so goal-supplied values override tt-Modify. When creating a new Modify row, default values are assigned as specified in the procedure.

# Source

- [[../Data Models/Modify/Modify - legacy]]
- progress-SLC/print/weigh-create-modify.p: procedure Create-Modify (DO TRANS delete b-tt-Modify; FIND/CREATE Modify by gTTProductCode; defaults; GlobalKillDateInEffect; ZuluCapture/SetZuluDept; buffer-copy; UpdateFromGoal when gProduceToWO)

# Impacted Systems

Modify table, tt-Modify (b-tt-modify), weigh-create-modify.p, weigh.w (calls Create-Modify).

# Traceability

- One tt-Modify per product session: Create-Modify deletes existing tt-Modify then builds one from Modify (and Goals when producing to WO).
- New Modify defaults: CasesPerPallet=0, CustName='', KillDateOffset=0, Lot='', ModText1='', ModText2='', OrdNum='', PdnOrdLineNum=0, PackDateOffset=0, PalletNum='', PalletTracking=no, Price=0, TareToUse=1, VarOffset=0.

# Assertions

- Before building tt-Modify, all existing b-tt-Modify rows are deleted in an exclusive transaction.
- Modify is found by Modify.ProductCode = gTTProductCode; if not available, CREATE Modify with the listed defaults.
- After applying GlobalKillDate and ZuluDept (see [[BR-Modify-002-GlobalKillDate-and-ZuluCapture]]), create b-tt-modify and buffer-copy Modify to b-tt-modify.
- When gProduceToWO is true, UpdateFromGoal is run (see [[BR-Modify-005-UpdateFromGoal-Goals-to-tt-Modify]]).

# Related Information

- [[BR-Modify-001-One-per-Product]]
- [[BR-Modify-002-GlobalKillDate-and-ZuluCapture]]
- [[BR-Modify-005-UpdateFromGoal-Goals-to-tt-Modify]]
