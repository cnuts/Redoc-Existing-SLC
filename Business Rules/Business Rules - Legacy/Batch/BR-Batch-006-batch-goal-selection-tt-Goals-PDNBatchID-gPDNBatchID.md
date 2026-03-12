---
Risk-Level: P0
Business-Rule-Id: BR-Batch-006
Deprecated: false
---

# Description

When the user selects "Produce to Goal" by batch (goals/w-select-batch-goals.w), the window builds a temp-table tt-Goals from BatchOrder and Goals. For each BatchOrder with the chosen BatchDate and ProdCode, it finds Goals that match by product, date, and either (OrdNum/OrdRefLine for order-linked batches) or (OrdNum empty and Description BEGINS "Batch:" + STRING(BatchOrder.BatchID) for batch goals). Each goal added to tt-Goals gets tt-Goals.PDNBatchID = BatchOrder.BatchID. The goal buttons are created with a name that includes GoalID and PDNBatchID (e.g. 'GOAL-' + STRING(tt-Goals.GoalID) + '-' + STRING(tt-Goals.PDNBatchID)). When the user chooses a goal button, the code parses vPDNBatchID = INT64(ENTRY(3, SELF:Name, "-")) and assigns gPDNBatchID = vPDNBatchID. When the user selects a fresh (non-batch) goal (goals/w-select-fresh-goals.w), gPDNBatchID is set to 0. Thus gPDNBatchID is set from the selected batch goal's BatchOrder.BatchID and cleared when producing to a fresh goal; createserial.p then assigns Serial.PDNBatchID = gPDNBatchID.

# Source

- progress-SLC/goals/w-select-batch-goals.w: FOR EACH BatchOrder WHERE BatchDate = ip-BatchDate AND ProdCode = ip-ProdCode: find Goals, create tt-Goals, tt-Goals.PDNBatchID = BatchOrder.BatchID. Button name 'GOAL-' + GoalID + '-' + PDNBatchID. ON CHOOSE: vPDNBatchID = ENTRY(3, SELF:Name, "-"), gPDNBatchID = vPDNBatchID.
- progress-SLC/goals/w-select-fresh-goals.w: gPDNBatchID = 0 when selecting fresh goal.

# Impacted Systems

BatchOrder, Goals, tt-Goals, gPDNBatchID (gprocessingvars.i), goals/w-select-batch-goals.w, goals/w-select-fresh-goals.w, lib/createserial.p (Serial.PDNBatchID).

# Traceability

- Button name encoding allows the UI to pass both GoalID and PDNBatchID so that the weigh flow receives the correct gPDNBatchID for the chosen batch goal.

# Assertions

- Only goals that match a BatchOrder row for the selected date and product are shown in the batch goal list; selecting one sets gPDNBatchID for the session until a fresh goal is chosen or flow is left.

# Related Information

- [[BR-Batch-003-Serial-PDNBatchID-from-batch-goal]]
