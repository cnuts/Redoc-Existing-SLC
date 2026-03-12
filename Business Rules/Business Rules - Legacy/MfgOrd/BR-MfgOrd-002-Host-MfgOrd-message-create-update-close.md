---
Risk-Level: P0
Business-Rule-Id: BR-MfgOrd-002
Deprecated: false
---

# Description

Host messages for manufacturing orders (msg-mfgord-update.p and msg-mfgord-single-update.p) accept JSON with an optional ERPPlantID and a MfgOrd payload. MfgOrd records are found by MfgOrdNum; if not found a new row is created, otherwise the existing row is updated. In the batch variant (msg-mfgord-update.p), any MfgOrd present in the database but not in the message is closed (MfgOrdStatus set to "Closed"). In the single variant (msg-mfgord-single-update.p), MfgOrdStatus is taken from the message and no close-missing step runs. Optional ERPPlantID in the message updates SiteConfigOption and ConfigOptionMaster for "ERPPlantID" (and appends to AcceptableValues if not already present).

# Source

- progress-SLC/host/msg-mfgord-update.p: JSON parse; optional ERPPlantID → SiteConfigOption/ConfigOptionMaster; tt-MfgOrd READ-JSON; FOR EACH tt-MfgOrd FIND MfgOrd by MfgOrdNum, CREATE or UPDATE (MfgOrdStatus = "Open"); FOR EACH MfgOrd not in tt-MfgOrd run ip-Close-MfgOrd (MfgOrdStatus = "Closed")
- progress-SLC/host/msg-mfgord-single-update.p: same create/update by MfgOrdNum; MfgOrdStatus from tt-MfgOrd.MfgOrdStatus; no close-missing loop

# Impacted Systems

MfgOrd table, SiteConfigOption, ConfigOptionMaster (ERPPlantID), host messaging, temp\msg-mfgord-update.log.

# Traceability

- MfgOrd key for lookup: MfgOrd.MfgOrdNum.
- New row: MfgOrdDate, MfgOrdNum, ProdCode from tt-MfgOrd; msg-mfgord-update sets MfgOrdStatus = "Open"; msg-mfgord-single-update sets MfgOrdStatus = tt-MfgOrd.MfgOrdStatus.
- Batch close: MfgOrd rows whose MfgOrdNum is not in the incoming tt-MfgOrd list are closed via ip-Close-MfgOrd.

# Assertions

- msg-mfgord-update.p: after applying message, every SLC.MfgOrd with no matching tt-MfgOrd.MfgOrdNum gets MfgOrdStatus = "Closed".
- msg-mfgord-single-update.p: MfgOrdStatus is not forced to "Open"; it is set from the message. No scan to close other MfgOrd rows.
- ERPPlantID: when non-blank, SiteConfigOption.OptionValue and ConfigOptionMaster are created/updated; AcceptableValues is pipe-separated and the new value is prepended if not already in the list.

# Related Information

- [[BR-MfgOrd-001-ProdCode-and-Serial-relationship]]
- [[BR-Config-001-OptionName-unique-and-SiteConfig]]
