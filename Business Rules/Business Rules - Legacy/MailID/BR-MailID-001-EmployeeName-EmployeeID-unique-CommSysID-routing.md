---
Risk-Level: P0
Business-Rule-Id: BR-MailID-001
Deprecated: false
---

# Description

MailID table stores EmployeeName, EmployeeID, EMailID, CommSysID, Descr for routing. Primary key is (EmployeeName, EmployeeID) unique (index EmplNameID). CommSysID links operators/locations to the communication system for message routing. Indexes exist on CommSysID, Descr, and EMailID for lookups.

# Source

- [[../Data Models/0.Legacy Database Schema]] — MailID (EmployeeName, EMailID, CommSysID, Descr, EmployeeID), index EmplNameID yes yes EmployeeName, EmployeeID
- [[../Data Models/Message/Message - legacy]] — §1 Overview (MailID: "Stores CommSysID and EmployeeName, EMailID... for routing")
- slc.df mailid

# Impacted Systems

MailID table; communication routing, inbox/messaging (CommHdr routing by SendTo/SentFrom), operator/location to CommSysID mapping.

# Traceability

- EmplNameID primary unique (EmployeeName, EmployeeID); CommSysID, Descr, Email indexes for lookups

# Assertions

- (EmployeeName, EmployeeID) is unique.
- MailID links operators/locations to CommSysID and EMailID for message routing over CFS network.

# Related Information

