---
Risk-Level: P0
Business-Rule-Id: BR-Item-003
Deprecated: false
---

# Description

The Create-TT-Item procedure (weigh-create-tt-item.p) builds the tt-Item temp-table for the current product. It deletes all existing tt-Item rows, then for each tt-ProductProcess row where Item is not unknown (?), finds or creates a tt-Item row by ItemCode (tt-ProductProcess.Item). If no tt-Item exists, it creates one, finds the Item table row by that ItemCode, and buffer-copies Item to tt-Item (with PctTareOverride and WgtTareOverride set to ?). When producing to work order (gProduceToWO), UpdateFromGoal is run for Goals into tt-Item only for the first such item (vUpdatedAnItem); the goals architecture does not support multiple items per product, so only the first item is updated from the goal.

# Source

- progress-SLC/print/weigh-create-tt-item.p: Create-TT-Item deletes all b-tt-Item; FOR EACH tt-ProductProcess WHERE Item <> ?: FIND b-tt-Item by ItemCode; IF NOT AVAIL create b-tt-Item, FIND Item by ItemCode, buffer-copy Item to b-tt-Item (PctTareOverride, WgtTareOverride = ?); if gProduceToWO and not vUpdatedAnItem run UpdateFromGoal, set vUpdatedAnItem = yes. UpdateFromGoal copies Goals fields to b-tt-Item when <> ? (DateAI, DateAIDate, Desc1/2, ExactItemsPerCaseRequired, TareWgt, ExtraTareType, Wgt/Pct overrides, ItemDateFormat, MinWgt, MaxWgt, Min/MaxLabelWgt, Offset2, PackType, PostTare, ProdDateFormat, SellByFormat/Offset, ShortDesc1/2/3, StdWgt, Text1–10, WgtRoundDigit, WgtRoundOrTruncate, VarFormat, WgtUnits)

# Impacted Systems

tt-Item, Item table, tt-ProductProcess, Goals, weigh-create-tt-item.p, weigh.w.

# Traceability

- tt-Item is populated only for Item codes that appear in tt-ProductProcess.Item for the current product.
- UpdateFromGoal runs at most once per Create-TT-Item call (first item only).

# Assertions

- When the same ItemCode appears in multiple tt-ProductProcess rows, only one tt-Item row is created (check "if not avail b-tt-item" before create).
- Goals fields override tt-Item only when gProduceToWO and vUpdatedAnItem is still false.

# Related Information

- [[BR-Item-001-ItemCode-range-and-schema]]
- [[BR-Item-002-ItemSerial-CaseSerialNum-GoalID]]
