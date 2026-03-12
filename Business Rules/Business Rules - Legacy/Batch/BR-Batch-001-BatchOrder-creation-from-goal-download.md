---
Risk-Level: P0
Business-Rule-Id: BR-Batch-001
Deprecated: false
---

# Description

BatchOrder rows are created when the host sends a Goal Download message (msg-goal-download.p) and a goal's Description matches "Batch:*" and OrdNum is null or blank. The code parses BatchID from Description, looks up BatchOrder by BatchID, BatchDate (= TgtCompDate), and ProdCode; if no BatchOrder row exists, it creates one with BatchID, BatchDate, ProdCode (OrdNum, OrdRefNum, OrdRefLine not set in this path).

# Source

- [[../Data Models/Batch/Batch - legacy]] — §4.1 How Batch Goals Arrive: msg-goal-download.p
- host/msg-goal-download.p

# Impacted Systems

BatchOrder table, Goals table, msg-goal-download.p, Produce to Goal by batch selection.

# Traceability

- [[../Data Models/0.Legacy Database Schema]] — BatchOrder (BatchID, BatchDate, OrdNum, OrdRefNum, OrdRefLine, ProdCode); index BatchDate primary
- Goal download creates BatchOrder so batch goals can be listed by date and product

# Assertions

- When goal Description matches "Batch:*" and OrdNum is null or blank, treat as batch goal.
- Parse BatchID from Description; ensure BatchOrder row exists for BatchID, BatchDate (TgtCompDate), ProdCode; create if missing.
- BatchOrder stores one row per batch known to the SLC; used for "Produce to Goal" by batch (date + product).

# Related Information

