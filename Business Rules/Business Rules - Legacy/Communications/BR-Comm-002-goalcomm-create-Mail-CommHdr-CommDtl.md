---
Risk-Level: P0
Business-Rule-Id: BR-Comm-002
Deprecated: false
---

# Description

goalcomm.p creates an internal mail-style message by creating one CommHdr and one CommDtl in the primary database. Inputs are ipSubject and ipText; output opResult is set to 0. CommHdr is created first: CommSeqID = NEXT-VALUE(CommSeqID), CommSeqIDThread = CommSeqID, DateSent = TODAY, Deleted = no, ICIProcessed = no, Origin = 'CTS', Priority = 1, SendTo = STRING(gOperator), SentFrom = STRING(gOperator), Subject = ipSubject, TimeSent = string(time,'hh:mm:ss'), Type = 'Mail', WasRead = no; DateRead and TimeRead remain ? and ?. CommDtl is then created: Body = ipText, CommSeqID = CommHdr.CommSeqID, DetailNumber = 1. Thus internal mail is addressed to and from the current operator; only one detail row is created per call.

# Source

- progress-SLC/print/goalcomm.p: CREATE CommHdr, ASSIGN CommSeqID = next-value(CommSeqID), Type = 'Mail', SendTo = vMailTo (string(gOperator)), SentFrom = vMailTo, Subject = ipSubject, etc. CREATE CommDtl, ASSIGN Body = ipText, CommSeqID = CommHdr.CommSeqID, DetailNumber = 1.

# Impacted Systems

CommHdr, CommDtl, print/goalcomm.p, callers that send internal mail (e.g. goal-related messages).

# Traceability

- CommSeqID is taken from the CommSeqID sequence; one header and one detail per invocation.

# Assertions

- Type is always 'Mail'; SendTo and SentFrom are the current operator; DetailNumber is 1.

# Related Information

- [[BR-Comm-001-CommDtl-CommHdr-ReadyToSend]]
