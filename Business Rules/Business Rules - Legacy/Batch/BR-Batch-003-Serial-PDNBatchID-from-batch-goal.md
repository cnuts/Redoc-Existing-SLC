---
Risk-Level: P0
Business-Rule-Id: BR-Batch-003
Deprecated: false
---

# Description

When a serial is created in lib/createserial.p, the code assigns pb-Serial.PDNBatchID = gPDNBatchID. Every case (serial) produced while gPDNBatchID is set (e.g. after selecting a batch goal) is stamped with that batch ID. When the user is not producing to a batch goal, gPDNBatchID is 0 (reset in main menu and in fresh-goal flow). The chosen goal's PDNBatchID is set from BatchOrder.BatchID and stored in gPDNBatchID when the user selects a goal by batch (date + product).

# Source

- [[../Data Models/Batch/Batch - legacy]] — §2.2 Serial.PDNBatchID, §2.1 Purpose, §4.2 Site Config
- lib/createserial.p; goals selection (w-select-batch-date.w, etc.)

# Impacted Systems

Serial table (PDNBatchID), BatchOrder, Goals, createserial.p, Produce to Goal by batch flow, gPDNBatchID.

# Traceability

- [[../Data Models/0.Legacy Database Schema]] — Serial.PDNBatchID (data/Serial-PDNBatchID.df)
- gPDNBatchID set from BatchOrder.BatchID when user picks batch goal; cleared when leaving or not choosing a goal

# Assertions

- Serial.PDNBatchID = gPDNBatchID at serial creation.
- gPDNBatchID set from BatchOrder.BatchID when user selects a batch goal (date + product); cleared when not producing to batch goal.
- PDNBatchID links the serial to a batch (e.g. from host or MII).

# Related Information

