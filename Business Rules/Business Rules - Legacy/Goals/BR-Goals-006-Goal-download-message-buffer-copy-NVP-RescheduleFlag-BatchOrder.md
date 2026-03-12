---
Risk-Level: P0
Business-Rule-Id: BR-Goals-006
Deprecated: false
---

# Description

The Goal Download message (msg-goal-download.p) creates or updates Goals from JSON (tt-Goals). Buffer-copy from tt-Goals to Goals excludes PkgDesc3Size, CurrentCount, CurrentLabelWgt, CurrentNetWgt, and Tolerance so the server does not overwrite local progress. Goals.DateStatusChange = TODAY. AllScaleTotalCount, AllScaleTotalNetWgt, AllScaleTotalLabelWgt are set from tt-Goals.CurrentCount, CurrentNetWgt, and CurrentLabelWgt respectively. Tolerance is set from tt-Goals.Tolerance, or 0 if ?. PkgDesc3Size is merged from tt-Goals NVP entries (CHR(2)-separated key=value): only entries whose name is not PrintedCount, PrintedWgt, Printed1, Printed2, or RescheduleFlag are applied (SLC keeps printed counts/weights). If tt-Goals.RescheduleFlag = "Yes", CurrentCount, CurrentLabelWgt, and CurrentNetWgt are set to 0 and PkgDesc3Size PrintedCount and PrintedWgt are set to "0". When tt-Goals.Description MATCHES "Batch:*" and OrdNum is ? or "", a BatchOrder row is created if missing: BatchID from first segment of the part after "Batch:" (before "-"), BatchDate = tt-Goals.TgtCompDate, ProdCode = tt-Goals.ProdCode.

# Source

- progress-SLC/host/msg-goal-download.p: READ-JSON to tt-Goals; FIND/CREATE Goals by GoalID; BUFFER-COPY tt-Goals EXCEPT PkgDesc3Size CurrentCount CurrentLabelWgt CurrentNetWgt Tolerance; ASSIGN DateStatusChange=TODAY, AllScaleTotal*=tt-Goals.Current*, Tolerance; PkgDesc3Size NVP loop excluding PrintedCount/PrintedWgt/Printed1/Printed2/RescheduleFlag; RescheduleFlag="Yes" zeroing; Description MATCHES "Batch:*" and OrdNum empty → CREATE BatchOrder by BatchID/TgtCompDate/ProdCode

# Impacted Systems

Goals table, msg-goal-download.p, PkgDesc3Size NVP, BatchOrder (SLC), host messaging.

# Traceability

- RM9453: CurrentCount from server goes to AllScaleTotalCount (not overwriting local CurrentCount).
- SLC97: BatchOrder creation when goal Description indicates batch and no order.

# Assertions

- Local PkgDesc3Size printed keys (PrintedCount, PrintedWgt, Printed1, Printed2) are never overwritten from message NVP; RescheduleFlag NVP is not stored in PkgDesc3Size.
- RescheduleFlag = "Yes" resets local counts and printed values for rescheduling.
- BatchOrder is created only when Description matches "Batch:*" and OrdNum is null or blank.

# Related Information

- [[BR-Goals-001-Status-values-and-transitions]]
- [[BR-Goals-005-Host-download-and-Counter-Goal]]
