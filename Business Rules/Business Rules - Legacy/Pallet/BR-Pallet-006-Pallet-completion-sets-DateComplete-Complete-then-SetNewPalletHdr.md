---
Risk-Level: P0
Business-Rule-Id: BR-Pallet-006
Deprecated: false
---

# Description

When the current pallet is completed (print/weigh.w), the application finds the current PalletHdr exclusive-lock and assigns PalletHdr.DateComplete = today and PalletHdr.Complete = yes, then runs SetNewPalletHdr so the next case is assigned to a new pallet. So completing a pallet marks the header with the completion date and flag, and immediately creates a new PalletHdr and updates tt-Modify.PalletNum to the new pallet number. This occurs in the normal flow when the operator completes the pallet (e.g. after printing the pallet label when CaseCount reached CasesPerPallet or when manually completing).

# Source

- progress-SLC/print/weigh.w — find current PalletHdr exclusive-lock. assign PalletHdr.DateComplete = today, PalletHdr.Complete = yes. find current PalletHdr no-lock. run SetNewPalletHdr. vPrintPalletRequested = no.

# Impacted Systems

PalletHdr (DateComplete, Complete), print/weigh.w, SetNewPalletHdr.

# Traceability

- [[BR-Pallet-001-PalletNum-format-and-unique]] — PalletHdr fields
- [[BR-Pallet-004-SetNewPalletHdr-creates-PalletHdr-and-updates-Modify-PalletNum]] — new pallet after completion
- [[BR-Pallet-005-Pallet-label-print-when-CaseCount-meets-CasesPerPallet-or-requested]] — completion often follows label print

# Assertions

- On pallet completion, PalletHdr.DateComplete and PalletHdr.Complete are set, then SetNewPalletHdr is run.
- The next case is assigned to the newly created pallet via tt-Modify.PalletNum.

# Related Information

