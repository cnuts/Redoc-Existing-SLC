---
Risk-Level: P0
Business-Rule-Id: BR-CrossRef-003
Deprecated: false
---

# Description

The procedure remove-update-crossref.p deletes every CrossRef row where Application = "UpdateAvailable". It runs with exclusive-lock on the matching rows. This provides a standalone way to clear the update-available flag (CrossRef entries used to signal that an update is available), consistent with the same delete performed at the start of the host update load in update-slc.w.

# Source

- progress-SLC/lib/remove-update-crossref.p: FOR EACH CrossRef EXCLUSIVE-LOCK WHERE CrossRef.Application = "UpdateAvailable": DELETE. RETURN.

# Impacted Systems

CrossRef table, lib/remove-update-crossref.p, callers that need to clear UpdateAvailable entries.

# Traceability

- Same condition as in update-slc.w (delete CrossRef where Application = "UpdateAvailable").
- No other CrossRef rows are affected.

# Assertions

- Only rows with Application = "UpdateAvailable" are deleted.
- Procedure performs no other table changes.

# Related Information

- [[BR-CrossRef-002-update-slc-crossref-d-import-UpdateAvailable-delete]]
