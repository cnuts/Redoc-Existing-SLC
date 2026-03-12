---
Risk-Level: P0
Business-Rule-Id: BR-Pallet-004
Deprecated: false
---

# Description

When a new pallet header is needed (print/weigh.w procedure SetNewPalletHdr), the procedure runs in a transaction: it creates a new PalletHdr, assigns PalletHdr.PalletNum = f-get-new-palletnum() and PalletHdr.CaseCount = 0, then finds tt-Modify exclusive-lock and sets tt-Modify.PalletNum = PalletHdr.PalletNum. So the current Modify (and thus the current product's pallet context) is updated to the new pallet number. SetNewPalletHdr is called (1) when PalletHdr is not available for the current tt-Modify.PalletNum (e.g. after StartNewProduct or when palletize is turned on and no pallet was assigned), and (2) after marking the current pallet complete (DateComplete = today, Complete = yes). PalletNum is supplied by f-get-new-palletnum(); the schema defines a Sequence PalletNum (0, 1, 0, 9999, yes) and format PlantID (99) + YDDD + 9999 sequence.

# Source

- progress-SLC/print/weigh.w — procedure SetNewPalletHdr: do transaction: create PalletHdr. assign PalletHdr.PalletNum = f-get-new-palletnum(), PalletHdr.CaseCount = 0. find current tt-Modify exclusive-lock. tt-Modify.PalletNum = PalletHdr.PalletNum. Called when "if not avail PalletHdr then run SetNewPalletHdr" and after setting PalletHdr.Complete = yes.

# Impacted Systems

PalletHdr, tt-Modify, Modify.PalletNum, print/weigh.w, f-get-new-palletnum (or sequence PalletNum).

# Traceability

- [[BR-Pallet-001-PalletNum-format-and-unique]] — PalletNum format and uniqueness; Sequence PalletNum
- [[BR-Modify-001-One-per-Product]] — Modify per Product; PalletNum on Modify

# Assertions

- New PalletHdr is created with CaseCount = 0 and PalletNum from f-get-new-palletnum().
- tt-Modify.PalletNum is set to the new PalletHdr.PalletNum so subsequent case prints use the new pallet.

# Related Information

