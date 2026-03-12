---
Risk-Level: P0
Business-Rule-Id: BR-Comm-001
Deprecated: false
---

# Description

CommDtl is the child of CommHdr and provides the message body; there can be multiple details per header. When all details have been added, ReadyToSend should be set = yes. CommSeqID comes from the CommSeqID sequence. Primary key of CommDtl is (CommSeqID, DetailNumber) unique.

# Source

- [[../Data Models/0.Legacy Database Schema]] — CommDtl (description), CommHdr; Sequences CommSeqID
- slc.df CommDtl, CommHdr

# Impacted Systems

CommHdr, CommDtl tables; message build and send logic over CFS network (Inven, Production, CTS DB's).

# Traceability

- CommDtl: CommSeqID (from sequence), DetailNumber, Body x(4096), Blob
- CommHdr: CommSeqID PK, DateRead, DateSent, Deleted, Priority, SendTo, SentFrom, Subject, WasRead, Type, etc.

# Assertions

- CommDtl.CommSeqID references CommHdr.CommSeqID (header-detail relationship).
- DetailIDX unique (CommSeqID, DetailNumber).
- When all details for a header are added, ReadyToSend set to yes (application logic).

# Related Information

