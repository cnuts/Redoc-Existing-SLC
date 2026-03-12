---
Risk-Level: P0
Business-Rule-Id: BR-MfgOrd-003
Deprecated: false
---

# Description

The Select Manufacturing Order Number dialog (select-mfgord.w) lists MfgOrd rows that are Open, for the current product (gProductCode), and with MfgOrdDate on or before the adjusted pack date (v-AdjustedPack). The list is ordered by MfgOrdDate descending then MfgOrdNum. The user must select exactly one row and choose Enter; the selected MfgOrdNum is then stored in CrossRef with Application = "LastsForProduct" and ID = "MfgOrdNum" (Descr = string of MfgOrdNum). If that CrossRef already exists (e.g. from a prior run), InitLastsFor skips the dialog and exits. If there are no qualifying MfgOrd rows, the dialog shows "No Manufacturing Orders exist for product code" and sets v-Quit so the flow exits.

# Source

- progress-SLC/print/select-mfgord.w: QUERY BROWSE-MfgOrd FOR EACH MfgOrd WHERE MfgOrd.MfgOrdStatus = "Open" AND MfgOrd.ProdCode = gProductCode AND MfgOrd.MfgOrdDate <= v-AdjustedPack NO-LOCK BY MfgOrd.MfgOrdDate DESCENDING BY MfgOrd.MfgOrdNum; btn-Enter: if NUM-SELECTED-ROWS <> 1 show "Please select a Manufacturing Order Number"; else CREATE CrossRef Application="LastsForProduct" ID="MfgOrdNum" Descr=STRING(MfgOrd.MfgOrdNum); InitLastsFor: if CrossRef LastsForProduct/MfgOrdNum exists then op-ExitNoSetupNeeded=YES; local-initialize: if NUM-RESULTS("Browse-MfgOrd")=0 then d-tsmsgbox "No Manufacturing Orders exist for product code", v-Quit=TRUE
- v-AdjustedPack from get-sellby-offset.p (BUFFER b-TT-Product, b-TT-Modify)

# Impacted Systems

MfgOrd table, CrossRef (LastsForProduct, MfgOrdNum), select-mfgord.w, weigh/print flow (gProductCode), get-sellby-offset.p.

# Traceability

- Only Open orders for the current product and with date <= adjusted pack date are selectable.
- Selected MfgOrdNum is persisted in CrossRef for use by CreateSerial (see [[BR-MfgOrd-004-Serial-MfgOrdNum-from-LastsForProduct-CrossRef]]).
- Skip when LastsForProduct/MfgOrdNum already set avoids re-prompting when operator has already chosen an order.

# Assertions

- Exactly one browse row must be selected on Enter; otherwise message "Please select a Manufacturing Order Number" and RETURN NO-APPLY.
- CrossRef record created/replaced with Descr = STRING(MfgOrd.MfgOrdNum) so downstream code can read INT64(Descr).
- If NUM-RESULTS("Browse-MfgOrd") = 0, user is notified and v-Quit is set; SetTimerToExit runs and dialog closes.

# Related Information

- [[BR-MfgOrd-001-ProdCode-and-Serial-relationship]]
- [[BR-MfgOrd-004-Serial-MfgOrdNum-from-LastsForProduct-CrossRef]]
- [[BR-CrossRef-001-Application-ID-unique]]
