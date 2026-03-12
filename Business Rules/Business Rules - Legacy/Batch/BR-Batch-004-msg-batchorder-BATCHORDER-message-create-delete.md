---
Risk-Level: P0
Business-Rule-Id: BR-Batch-004
Deprecated: false
---

# Description

The host BATCHORDER message (msg-batchorder.p) receives a JSON body and applies creates or deletes to the BatchOrder table. The message body is parsed into temp-table tt-BatchOrder, which is like SLC.BatchOrder with an extra field DeleteBatchOrder (logical). For each tt-BatchOrder row, the code finds SLC.BatchOrder where BatchDate, BatchID, ProdCode, OrdNum, OrdRefLine, and OrdRefNum all match. If such a row exists and tt-BatchOrder.DeleteBatchOrder = TRUE, it runs ip-Delete-BatchOrder (which finds the BatchOrder by recid under exclusive-lock and deletes it). If no matching BatchOrder exists and tt-BatchOrder.DeleteBatchOrder = FALSE, it creates SLC.BatchOrder and buffer-copies tt-BatchOrder to it. Thus the host can create new BatchOrder rows or delete existing ones by sending BATCHORDER message data; the match key is (BatchDate, BatchID, ProdCode, OrdNum, OrdRefLine, OrdRefNum).

# Source

- progress-SLC/host/msg-batchorder.p: TEMP-TABLE tt-BatchOrder LIKE SLC.BatchOrder FIELD DeleteBatchOrder AS LOGICAL. READ-JSON from ip-MsgBody. FOR EACH tt-BatchOrder: FIND SLC.BatchOrder WHERE BatchDate, BatchID, ProdCode, OrdNum, OrdRefLine, OrdRefNum match. IF AVAILABLE AND tt-BatchOrder.DeleteBatchOrder THEN RUN ip-Delete-BatchOrder(RECID). ELSE IF NOT AVAILABLE AND NOT DeleteBatchOrder THEN CREATE SLC.BatchOrder, BUFFER-COPY tt-BatchOrder TO SLC.BatchOrder. ip-Delete-BatchOrder: FIND by recid exclusive, DELETE.

# Impacted Systems

BatchOrder table, host/msg-batchorder.p, messaging-host.w (WHEN "BATCHORDER"), host/ERP or MII sending BATCHORDER messages.

# Traceability

- Match key includes OrdNum, OrdRefLine, OrdRefNum so order-linked batches are matched; goal-download-created rows (OrdNum/OrdRefLine/OrdRefNum not set) can still be matched or created via this message if the JSON supplies those values.

# Assertions

- No update path: existing rows are either left alone, or deleted when DeleteBatchOrder = TRUE. New rows are only created when no match and DeleteBatchOrder = FALSE.

# Related Information

- [[BR-Batch-001-BatchOrder-creation-from-goal-download]]
- [[BR-Batch-002-BatchOrder-purge-by-clear-days]]
