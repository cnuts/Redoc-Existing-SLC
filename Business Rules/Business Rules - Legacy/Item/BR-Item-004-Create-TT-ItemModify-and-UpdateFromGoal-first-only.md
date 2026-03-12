---
Risk-Level: P0
Business-Rule-Id: BR-Item-004
Deprecated: false
---

# Description

The Create-TT-ItemModify procedure (weigh-create-tt-itemmodify.p) builds the tt-ItemModify temp-table from ProductProcess items. It deletes all existing tt-ItemModify rows, then for each tt-ProductProcess row where Item is not unknown (?), finds ItemModify by ItemCode (tt-ProductProcess.Item); if no ItemModify exists it creates one with that ItemCode. It then finds or creates b-tt-ItemModify by ItemCode; if not found it creates b-tt-ItemModify and buffer-copies ItemModify to it. When gProduceToWO is true, UpdateFromGoal (Goals to b-tt-ItemModify) is run only for the first such item (vUpdatedAnItem). UpdateFromGoal copies CustName, OrdNum, PdnOrdLineNum, PackDateOffset, Price, and VarOffset from Goals when the goal value is not ?.

# Source

- progress-SLC/print/weigh-create-tt-itemmodify.p: Create-TT-ItemModify deletes all tt-ItemModify; FOR EACH tt-ProductProcess WHERE Item <> ?: FIND ItemModify by ItemCode; IF NOT AVAIL CREATE ItemModify ItemCode = tt-ProductProcess.Item; FIND b-tt-ItemModify by ItemCode; IF NOT AVAIL create b-tt-ItemModify, buffer-copy ItemModify to b-tt-ItemModify; if gProduceToWO and not vUpdatedAnItem run UpdateFromGoal, vUpdatedAnItem = yes. UpdateFromGoal: Goals.CustomerName→CustName, OrdNum, OrdRefLine→PdnOrdLineNum, PackDateOffset, UnitPrice→Price, PkgVarOffset→VarOffset when <> ?

# Impacted Systems

tt-ItemModify, ItemModify table, tt-ProductProcess, Goals, weigh-create-tt-itemmodify.p.

# Traceability

- ItemModify is created on demand by ItemCode when missing; tt-ItemModify is then populated from ItemModify and optionally from Goals (first item only).

# Assertions

- Only one Goals-based update is applied per Create-TT-ItemModify run (first item with gProduceToWO).
- Buffer-copy from ItemModify to b-tt-ItemModify occurs when creating a new tt-ItemModify row.

# Related Information

- [[BR-Item-001-ItemCode-range-and-schema]]
- [[BR-Item-003-Create-TT-Item-from-ProductProcess-UpdateFromGoal-first-only]]
