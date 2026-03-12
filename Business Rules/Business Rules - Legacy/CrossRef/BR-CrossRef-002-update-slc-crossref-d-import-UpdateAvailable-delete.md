---
Risk-Level: P0
Business-Rule-Id: BR-CrossRef-002
Deprecated: false
---

# Description

When the host update process (update-slc.w) runs, it first deletes all CrossRef rows where Application = "UpdateAvailable". Then when processing the file "crossref.d", for each imported record it creates tt-CrossRef, IMPORT tt-CrossRef, finds CrossRef by Application = tt-CrossRef.Application and ID = tt-CrossRef.ID under exclusive-lock. If no row exists it creates one. It then buffer-copies tt-CrossRef to the CrossRef table row and releases. Thus CrossRef rows are created or updated by (Application, ID) from crossref.d during host sync; and any existing "UpdateAvailable" CrossRef entries are cleared before loading table updates.

# Source

- progress-SLC/host/update-slc.w: Before loading: FOR EACH CrossRef EXCLUSIVE-LOCK WHERE CrossRef.Application = "UpdateAvailable": DELETE. WHEN "crossref.d": CREATE tt-CrossRef, IMPORT tt-CrossRef; FIND FIRST CrossRef WHERE Application = tt-CrossRef.Application AND ID = tt-CrossRef.ID exclusive no-error; IF NOT AVAILABLE THEN CREATE CrossRef; BUFFER-COPY tt-CrossRef TO CrossRef; RELEASE CrossRef

# Impacted Systems

CrossRef table, host/update-slc.w, tt-CrossRef, crossref.d, UpdateAvailable application usage.

# Traceability

- Matching is by (Application, ID); one row per pair in crossref.d. No delete of other CrossRef rows during this import—only create or update.
- "UpdateAvailable" is used elsewhere to signal that an update is available; clearing it is part of the update process.

# Assertions

- Rows with Application = "UpdateAvailable" are deleted at the start of the update load, not by the crossref.d import block.
- Import key is (Application, ID); existing row with same key is overwritten by buffer-copy.

# Related Information

- [[BR-CrossRef-001-Application-ID-unique]]
