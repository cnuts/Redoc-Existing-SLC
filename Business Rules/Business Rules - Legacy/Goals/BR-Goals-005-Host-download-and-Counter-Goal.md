---
Risk-Level: P0
Business-Rule-Id: BR-Goals-005
Deprecated: false
---

# Description

Goals are created or updated from the host via Goal Download messages (msg-goal-download.p). On download, buffer-copy from tt-Goals to Goals except PkgDesc3Size, CurrentCount, CurrentLabelWgt, CurrentNetWgt so server does not overwrite local progress. Goals.DateStatusChange = TODAY. When creating a serial for a goal, the system finds or creates CrossRef where Application = "Counter-Goal" and ID = STRING(Goals.GoalID); Descr holds a counter incremented on add and decremented on delete (e.g. for case number within goal). When a goal is purged, the corresponding Counter-Goal CrossRef row is deleted.

# Source

- [[../Data Models/Goal/Goal - Legacy]] — §4 Host Goal Download, §7 Serial and ItemSerial GoalID, §8 Counter-Goal CrossRef, §13 Goals Purge
- lib/createserial.p; clearlogs.p

# Impacted Systems

Goals table, host/msg-goal-download.p, CrossRef (Counter-Goal), createserial.p, clearlogs.p.

# Traceability

- PkgDesc3Size: SLC maintains PrintedCount, PrintedWgt, Printed1, Printed2; server sends other NVP keys
- Counter-Goal CrossRef used for serial numbering/sequencing within goal

# Assertions

- Goal download: create or update Goals; do not overwrite local PkgDesc3Size printed counts/weights or Current*/AllScaleTotal* in a way that loses SLC progress where documented.
- Counter-Goal CrossRef row exists per active goal; deleted when goal purged.
- Serial.GoalID and ItemSerial.GoalID set from gGoalsRowid when available in createserial.p/createitemserial.p.

# Related Information

