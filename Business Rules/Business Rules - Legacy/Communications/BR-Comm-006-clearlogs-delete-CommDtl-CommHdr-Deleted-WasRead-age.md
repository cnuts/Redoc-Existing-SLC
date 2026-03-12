---
Risk-Level: P0
Business-Rule-Id: BR-Comm-006
Deprecated: false
---

# Description

clearlogs.p deletes old communication records in two steps. It iterates over CommHdr where (CommHdr.Deleted OR CommHdr.WasRead) AND (TODAY - CommHdr.DateSent) > v-clear-serial-days. For each such header, it first deletes all CommDtl rows of that CommHdr (FOR EACH CommDtl OF CommHdr exclusive-lock: DELETE CommDtl). Then in a transaction it finds the CommHdr by rowid under exclusive-lock and deletes it, after optionally writing a message to the log (program-name, "Deleted CommHdr", CommSeqID, DateSent). Thus details are deleted before their header; only headers that are either Deleted or WasRead and older than v-clear-serial-days are purged.

# Source

- progress-SLC/lib/clearlogs.p: FOR EACH CommHdr NO-LOCK WHERE (Deleted OR WasRead) AND (today - DateSent) > v-clear-serial-days: 4000-CommDtl-Trans: FOR EACH CommDtl OF CommHdr exclusive-lock: DELETE CommDtl. 5000-CommHdr-Trans: FIND b-CommHdr by rowid exclusive, DELETE b-CommHdr, message "Deleted CommHdr" + CommSeqID + DateSent.

# Impacted Systems

CommHdr, CommDtl, lib/clearlogs.p, v-clear-serial-days (clear-logs day threshold).

# Traceability

- Deletion order is CommDtl then CommHdr to satisfy referential integrity (detail before header).

# Assertions

- Unread (WasRead = no) and not Deleted headers are not deleted by this logic; they are retained until marked read or deleted and then aged past the threshold.

# Related Information

- [[BR-Comm-001-CommDtl-CommHdr-ReadyToSend]]
