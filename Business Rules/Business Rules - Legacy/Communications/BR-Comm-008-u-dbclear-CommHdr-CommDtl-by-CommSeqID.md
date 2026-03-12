---
Risk-Level: P0
Business-Rule-Id: BR-Comm-008
Deprecated: false
---

# Description

The database clear utility (system/u-dbclear.w) includes CommHdr and CommDtl in its clear-down process. It invokes the shared include lib/dbclear-table-inproc.i twice: once with &tbl="CommHdr" and &fld="CommSeqID", and once with &tbl="CommDtl" and &fld="CommSeqID", with &skipvalues=" ". The include implements table-specific delete logic keyed by the given field. Thus both tables can be cleared via the utility; the key field used for the clear is CommSeqID. Variables vCommDtlDeleted and vCommHdrDeleted are defined in u-dbclear.w for counting or reporting deleted records.

# Source

- progress-SLC/system/u-dbclear.w: def var vCommDtlDeleted, vCommHdrDeleted. {lib/dbclear-table-inproc.i &tbl="CommHdr" &fld="CommSeqID" &skipvalues=" "}. {lib/dbclear-table-inproc.i &tbl="CommDtl" &fld="CommSeqID" &skipvalues=" "}.

# Impacted Systems

CommHdr, CommDtl, system/u-dbclear.w, lib/dbclear-table-inproc.i.

# Traceability

- Clearing order (if enforced in the include or caller) should delete CommDtl before CommHdr when both are cleared, to respect the header-detail relationship.

# Assertions

- The utility provides a systematic way to clear communication tables by CommSeqID; actual delete order depends on dbclear-table-inproc.i implementation.

# Related Information

- [[BR-Comm-001-CommDtl-CommHdr-ReadyToSend]]
- [[BR-Comm-006-clearlogs-delete-CommDtl-CommHdr-Deleted-WasRead-age]]
