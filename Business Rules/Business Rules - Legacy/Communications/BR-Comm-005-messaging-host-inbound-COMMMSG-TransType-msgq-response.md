---
Risk-Level: P0
Business-Rule-Id: BR-Comm-005
Deprecated: false
---

# Description

When the host receives a COMMMSG message (messaging-host.w), it parses the message body as JSON into tt-CH and tt-CD. For each tt-CH where SendTo = gICICTSID + STRING(gScaleID,"99") (messages addressed to the current scale), it processes as follows. It obtains a new CommSeqID via NEXT-VALUE(CommSeqID), repeating until no existing tt-CH row has that CommSeqID (to avoid collision with incoming data). It searches for host\msgq<TransType>.r (e.g. msgq6044.r). If found, it runs that procedure with (input original CommSeqID, input new CommSeqID, output v-Return). If not found, it creates a response CommHdr (b-CommH) with the new CommSeqID, SendTo = tt-CH.SentFrom, SentFrom = tt-CH.SendTo, ICIProcessed = YES, WasRead = NO, and buffer-copies tt-CH to b-CommH except CommSeqID, SendTo, SentFrom, ICIProcessed, WasRead; then creates one CommDtl with the new CommSeqID, DetailNumber = 1, Body = "Program not found.". After processing, it deletes the tt-CH and tt-CD rows for the original CommSeqID. Thus inbound messages to this scale are dispatched by TransType; missing handlers produce a "Program not found." response; processing consumes the temp-table rows for that message.

# Source

- progress-SLC/host/messaging-host.w: WHEN "COMMMSG": READ-JSON into dsComms. FOR EACH tt-CH WHERE SendTo = gICICTSID + scale BREAK BY CommSeqID: v-NewCommSeqID = NEXT-VALUE(CommSeqID); REPEAT WHILE existing tt-CH has that id get next. vStr = SEARCH("host\msgq" + TransType + ".r"). IF found RUN VALUE(vStr)(tt-CH.CommSeqID, v-NewCommSeqID, v-Return). ELSE CREATE b-CommH, b-CommD with Body "Program not found.". DELETE tt-CD for this CommSeqID, DELETE tt-CH for this CommSeqID.

# Impacted Systems

CommHdr, CommDtl, host/messaging-host.w, host/msgq*.r procedures, tt-CH, tt-CD.

# Traceability

- Response messages use a new CommSeqID; SendTo/SentFrom are swapped for the reply; ICIProcessed = YES on the created response.

# Assertions

- Only messages with SendTo matching the current scale (gICICTSID + gScaleID) are processed; each message is processed once and then removed from tt-CH/tt-CD.

# Related Information

- [[BR-Comm-001-CommDtl-CommHdr-ReadyToSend]]
