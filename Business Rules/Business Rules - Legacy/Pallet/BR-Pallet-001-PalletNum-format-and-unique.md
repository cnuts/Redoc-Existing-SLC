---
Risk-Level: P0
Business-Rule-Id: BR-Pallet-001
Deprecated: false
---

# Description

PalletNum format is PlantID (99) + YDDD + 9999 sequence (e.g. "99YDDD0001"). PalletHdr.PalletNum is the primary key (unique). PalletDtl has composite primary key (PalletNum, SerialNum) unique; each Serial can appear on at most one pallet detail per PalletNum. Sequence PalletNum (0, 1, 0, 9999, yes) cycles.

# Source

- [[../Data Models/0.Legacy Database Schema]] — PalletHdr, PalletDtl, Modify (PalletNum), Serial (PalletNum); Sequences PalletNum
- [[../Data Models/Serial/Serial - legacy]] — PalletNum, PalletHdr/PalletDtl relationship

# Impacted Systems

PalletHdr, PalletDtl, Serial.PalletNum, Modify.PalletNum, pallet creation and completion flows.

# Traceability

- PalletHdr index PalletNum unique; PalletDtl index PalletSerial unique (PalletNum, SerialNum)
- Serial.PalletNum set when case packed to pallet; PalletDtl links SerialNum to PalletNum

# Assertions

- PalletNum character X(10), format PlantID + YDDD + sequence.
- PalletHdr: one row per PalletNum; CaseCount, Complete, DateComplete, TimeComplete, Sent, DateSent, TimeSent.
- PalletDtl: (PalletNum, SerialNum) unique; ProductCode, LabelWgt, NetWgt per detail.
- When PalletNum specified on Serial, it must exist in PalletHdr.

# Related Information

