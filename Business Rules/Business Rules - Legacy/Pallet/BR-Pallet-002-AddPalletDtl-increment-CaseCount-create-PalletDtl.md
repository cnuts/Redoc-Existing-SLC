---
Risk-Level: P0
Business-Rule-Id: BR-Pallet-002
Deprecated: false
---

# Description

When adding a serial to a pallet (pallets/addpalletdtl.p), the procedure runs in a transaction: it finds the current PalletHdr exclusive-lock, increments PalletHdr.CaseCount by 1, creates a new PalletDtl, and assigns PalletDtl.PalletNum = PalletHdr.PalletNum, PalletDtl.SerialNum = Serial.SerialNum, PalletDtl.ProductCode = Serial.ProductCode, PalletDtl.LabelWgt = Serial.LabelWgt, PalletDtl.NetWgt = Serial.NetWgt. So each added case increments the header CaseCount and creates one detail row linking that Serial to the pallet. Run by print/case-print.p (and default-case-print.p) when PalletHdr is available (i.e. when pallet tracking is in effect and a current pallet exists).

# Source

- progress-SLC/pallets/addpalletdtl.p — do transaction: find current PalletHdr exclusive-lock. PalletHdr.CaseCount = PalletHdr.CaseCount + 1. create PalletDtl. assign PalletDtl.LabelWgt = Serial.LabelWgt, PalletDtl.NetWgt = Serial.NetWgt, PalletDtl.PalletNum = PalletHdr.PalletNum, PalletDtl.ProductCode = Serial.ProductCode, PalletDtl.SerialNum = Serial.SerialNum.
- progress-SLC/print/case-print.p — if avail PalletHdr then run pallets/addpalletdtl.p (buffer PalletHdr, buffer Serial).

# Impacted Systems

PalletHdr, PalletDtl, Serial, pallets/addpalletdtl.p, print/case-print.p.

# Traceability

- [[BR-Pallet-001-PalletNum-format-and-unique]] — PalletDtl (PalletNum, SerialNum) unique; PalletHdr.CaseCount
- [[BR-Pallet-004-SetNewPalletHdr-creates-PalletHdr-and-updates-Modify-PalletNum]] — PalletHdr must exist before adding detail

# Assertions

- AddPalletDtl is invoked only when PalletHdr is available (pallet tracking and current pallet set).
- One PalletDtl row is created per call; PalletHdr.CaseCount is incremented by 1 in the same transaction.

# Related Information

