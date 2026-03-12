---
Risk-Level: P0
Business-Rule-Id: BR-Pallet-003
Deprecated: false
---

# Description

When removing a serial from a pallet (pallets/cancelpalletdtl.p), the procedure finds PalletDtl by ipSerial (SerialNum). If no PalletDtl is found, it returns without action. Otherwise, in a transaction it finds the PalletHdr for that PalletDtl.PalletNum (exclusive-lock), and if the header is available it deletes the PalletDtl and decrements PalletHdr.CaseCount by 1. So canceling a case from a pallet removes its detail row and decrements the header case count. Run by btnCancel in print/s-printlbl.w (CancelCase) and by btnRemove in pallets/d-palletdet.w. Comment in source: "If hdr already sent, don't bother" — the current code does not check PalletHdr.Sent before performing the delete and decrement.

# Source

- progress-SLC/pallets/cancelpalletdtl.p — find first PalletDtl where PalletDtl.SerialNum = ipSerial no-error. if not avail PalletDtl then return. do transaction: find first pb-PalletHdr where pb-PalletHdr.PalletNum = PalletDtl.PalletNum exclusive-lock no-error. if avail pb-PalletHdr then delete PalletDtl, pb-PalletHdr.CaseCount = pb-PalletHdr.CaseCount - 1.

# Impacted Systems

PalletDtl, PalletHdr, pallets/cancelpalletdtl.p, s-printlbl.w (CancelCase), pallets/d-palletdet.w (btnRemove).

# Traceability

- [[BR-Pallet-001-PalletNum-format-and-unique]] — PalletDtl (PalletNum, SerialNum)
- [[BR-Pallet-002-AddPalletDtl-increment-CaseCount-create-PalletDtl]] — inverse operation

# Assertions

- If no PalletDtl exists for the given SerialNum, the procedure returns without error.
- When PalletHdr is found, PalletDtl is deleted and CaseCount is decremented in one transaction.

# Related Information

