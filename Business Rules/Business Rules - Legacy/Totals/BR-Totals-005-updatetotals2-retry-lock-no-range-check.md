---
Risk-Level: P0
Business-Rule-Id: BR-Totals-005
Deprecated: false
---

# Description

lib/updatetotals2.p updates Totals (and Goals) for serial add/delete but differs from lib/updatetotals.p in two ways. (1) It does not perform the ProductCode, LabelWgt, or NetWgt range checks at entry; those checks are only in updatetotals.p (see [[BR-Serial-012-UpdateTotals-range-checks-ProductCode-LabelWgt-NetWgt]]). (2) To obtain an exclusive lock on the Totals row, it retries up to 10 times, waiting 1 second between attempts (REPEAT WHILE NOT AVAILABLE(Totals) AND vRetryCount < 10). If Totals is still not available after the retries, it runs ip-Logger("Unable to Update Totals due to Record Lock.") and does not update Totals; po-RetVal is not set to a specific error string in that path. So callers using updatetotals2.p do not get a "No totals update" or range-error return value when the lock fails.

# Source

- progress-SLC/lib/updatetotals2.p — FIND FIRST Totals … EXCLUSIVE-LOCK NO-WAIT NO-ERROR. vRetryCount = 0. REPEAT WHILE NOT AVAILABLE(Totals) AND vRetryCount < 10: (wait 1 second), FIND again, vRetryCount + 1. IF AVAILABLE(Totals) THEN DO (update). ELSE DO: RUN ip-Logger(INPUT "Unable to Update Totals due to Record Lock."). END.

# Impacted Systems

Totals table, lib/updatetotals2.p, callers that invoke updatetotals2.p (no pre-validation of serial weights/code).

# Traceability

- [[BR-Totals-002-UpdateTotals-lock-timeout-No-totals-update]] — updatetotals.p uses 15 sec wait and returns "No totals update"
- [[BR-Serial-012-UpdateTotals-range-checks-ProductCode-LabelWgt-NetWgt]] — range checks only in updatetotals.p

# Assertions

- updatetotals2.p does not validate ProductCode, LabelWgt, or NetWgt ranges before updating Totals.
- updatetotals2.p retries exclusive lock on Totals up to 10 times with 1 second between attempts; on persistent lock failure it logs and skips the update.

# Related Information

