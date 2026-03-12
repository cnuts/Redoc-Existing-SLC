---
Risk-Level: P0
Business-Rule-Id: BR-Goals-003
Deprecated: false
---

# Description

Goals are purged solely by age (DateStatusChange), regardless of status, after the configured number of days. Site config option Goals-Purge-Days (default 2) controls the retention. For each Goals where (TODAY - Goals.DateStatusChange) > v-goals-clear-days, the system deletes the Counter-Goal CrossRef row and then deletes the Goals row.

# Source

- [[../Data Models/Goal/Goal - Legacy]] — §13 Goals Purge (clearlogs.p)
- SiteConfigOption Goals-Purge-Days; ConfigOptionMaster description "Delete Goals if older than date, disregard status"

# Impacted Systems

Goals table, CrossRef (Application = "Counter-Goal"), lib/clearlogs.p, SiteConfigOption.

# Traceability

- clearlogs.p; Create-SiteConfig for Goals-Purge-Days if not present (value 2)

# Assertions

- v-goals-clear-days = INT(SiteConfigOption.OptionValue) for Goals-Purge-Days.
- Purge condition: (TODAY - Goals.DateStatusChange) > v-goals-clear-days.
- Before deleting Goals: delete CrossRef where Application = "Counter-Goal" and ID = STRING(Goals.GoalID).
- Orphan Counter-Goal CrossRef rows can be cleaned up after purge.

# Related Information

