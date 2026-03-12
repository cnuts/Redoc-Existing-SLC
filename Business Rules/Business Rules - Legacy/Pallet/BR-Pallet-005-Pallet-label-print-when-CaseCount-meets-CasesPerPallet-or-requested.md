---
Risk-Level: P0
Business-Rule-Id: BR-Pallet-005
Deprecated: false
---

# Description

The pallet label is printed (print/pallet-print.p) when all of the following hold: (1) b-tt-modify.PalletTracking is true, (2) either PalletHdr.CaseCount >= b-tt-modify.CasesPerPallet or ipPrintPalletRequested is true. When both conditions are met, pallets/palletlabel.p is run for each repetition (b-tt-ProductProcess.Repetitions). palletlabel.p calls pallets/ord-plt-lbl.p with gPlantID, PalletHdr.PalletNum, and empty print destination, then sends the returned label string to the output device (WriteToDevice in gh_SendOutput). So pallet tracking is driven by Modify.PalletTracking and Modify.CasesPerPallet; the label is printed when the case count reaches the configured cases per pallet or when a manual print is requested.

# Source

- progress-SLC/print/pallet-print.p — if avail b-tt-modify and b-tt-modify.PalletTracking then if PalletHdr.CaseCount >= b-tt-modify.CasesPerPallet or ipPrintPalletRequested then do vCtr = 1 to b-tt-ProductProcess.Repetitions: run pallets/palletlabel.p (input 'A', input ipCTLOutputDevice, input b-tt-ProductProcess.Device, output vMessage, buffer PalletHdr).
- progress-SLC/pallets/palletlabel.p — run pallets/ord-plt-lbl.p (gPlantID, pb-PalletHdr.PalletNum, '', output vOutString); then WriteToDevice with vOutString.

# Impacted Systems

Modify.PalletTracking, Modify.CasesPerPallet, PalletHdr.CaseCount, print/pallet-print.p, pallets/palletlabel.p, pallets/ord-plt-lbl.p, b-tt-ProductProcess.Repetitions.

# Traceability

- [[BR-Pallet-001-PalletNum-format-and-unique]] — PalletHdr
- [[BR-Pallet-002-AddPalletDtl-increment-CaseCount-create-PalletDtl]] — CaseCount incremented when cases added
- [[BR-Modify-001-One-per-Product]] — Modify per Product

# Assertions

- Pallet label is printed only when PalletTracking is on and (CaseCount >= CasesPerPallet or print requested).
- Label content is built by ord-plt-lbl.p from PalletHdr and PalletDtl/Serial data; output is sent to the configured device.

# Related Information

