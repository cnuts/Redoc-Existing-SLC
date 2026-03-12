---
Risk-Level: P0
Business-Rule-Id: BR-Comm-004
Deprecated: false
---

# Description

The host messaging process (messaging-host.w) sends outbound CommHdr/CommDtl to the message queue as follows. It selects CommHdr rows where WasRead = NO. For each, under exclusive-lock (NO-WAIT), it buffer-copies the CommHdr to tt-CH and each related CommDtl (CommDtl.CommSeqID = CommHdr.CommSeqID) to tt-CD. It builds JSON from dataset dsComms (tt-CH, tt-CD) and sends it with MESSAGETYPE SLCCOMMS via f-Msg-Queue-SendMsg. When the send returns TRUE, it sets WasRead = YES on the corresponding CommHdr row(s) in the database. Up to 10 records are batched before sending; any remainder is sent in a final send. After sending, temp-tables tt-CH and tt-CD are emptied. Thus only unread (WasRead = NO) headers are sent; success is acknowledged by marking them WasRead = YES.

# Source

- progress-SLC/host/messaging-host.w: FOR EACH CommHdr NO-LOCK WHERE WasRead = NO; FIND b-SLCCommHdr EXCLUSIVE-LOCK NO-WAIT; if avail BUFFER-COPY to tt-CH, FOR EACH CommDtl of CommHdr BUFFER-COPY to tt-CD. When v-RecsToSend >= 10 or after loop, WRITE-JSON, f-Msg-Queue-SendMsg; IF v-RETURN THEN for each tt-CH set b-SLCCommHdr.WasRead = YES. EMPTY tt-CH, tt-CD.

# Impacted Systems

CommHdr, CommDtl, host/messaging-host.w, message queue (f-Msg-Queue-SendMsg), tt-CH, tt-CD, dsComms.

# Traceability

- Batch size 10; send format MESSAGETYPE: SLCCOMMS, MESSAGEDATA: <JSON><EOM>.

# Assertions

- WasRead is set to YES only after a successful send; rows that fail to send remain WasRead = NO for retry.

# Related Information

- [[BR-Comm-001-CommDtl-CommHdr-ReadyToSend]]
