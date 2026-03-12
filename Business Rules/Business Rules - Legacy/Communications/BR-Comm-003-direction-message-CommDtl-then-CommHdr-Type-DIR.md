---
Risk-Level: P0
Business-Rule-Id: BR-Comm-003
Deprecated: false
---

# Description

Direction messages to Inven (e.g. current product code) create CommDtl first, then CommHdr, in a single transaction, using the same CommSeqID. msg6044-curr-prodcode-create.p is an example: it gets v_CommSeqID = NEXT-VALUE(CommSeqID), creates CommDtl with that CommSeqID, Body set via NVP (e.g. ProdCode), DetailNumber = 1; then creates CommHdr with the same CommSeqID, Type = "DIR", Subject = "6044", TransType = 6044, SendTo = "WPL" + STRING(gScaleID,"99"), SentFrom = gICICTSID + STRING(gScaleID,'99'), Origin = SentFrom, ICIProcessed = no, WasRead = no, DateSent = TODAY, TimeSent = STRING(TIME,'HH:MM:SS'), CommSeqIDThread = 0. Thus direction messages target "WPL" plus scale and identify the sending scale in Origin/SentFrom; creation order is detail then header within one transaction.

# Source

- progress-SLC/print/msg6044-curr-prodcode-create.p: DO TRANS: v_CommSeqID = NEXT-VALUE(CommSeqID). CREATE CommDtl, ASSIGN CommSeqID = v_CommSeqID, Body = f-NVP-set-value(..., "ProdCode", ...), DetailNumber = 1. CREATE CommHdr, ASSIGN CommSeqID = v_CommSeqID, Type = "DIR", Subject = "6044", TransType = 6044, SendTo = "WPL" + scale, SentFrom = Origin = gICICTSID + scale, etc.

# Impacted Systems

CommHdr, CommDtl, print/msg6044-curr-prodcode-create.p, op-send-dir-new-prodcode.p, direction message flow to Inven.

# Traceability

- Same CommSeqID ties header and detail; TransType identifies the message type for host processing (msgq6044.r etc.).

# Assertions

- For this pattern, CommDtl is created before CommHdr in the same transaction; Type = "DIR"; SendTo format "WPL" + scale number.

# Related Information

- [[BR-Comm-001-CommDtl-CommHdr-ReadyToSend]]
