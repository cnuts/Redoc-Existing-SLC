---
Risk-Level: P0
Business-Rule-Id: BR-Batch-002
Deprecated: false
---

# Description

BatchOrder rows are purged by lib/clearlogs.p using site config "BatchOrder Clear Days" (default 7). For each BatchOrder with BatchDate < TODAY - v-BatchOrder-Clear-Days, the system checks that no Goals reference that batch (by OrdNum/OrdRefLine or ProdCode/TgtCompDate/BatchDate); only then does it DELETE the BatchOrder. Batch records are retained only for the configured window and only if not still referenced by goals.

# Source

- [[../Data Models/Batch/Batch - legacy]] — §2.1 BatchOrder Table (Maintenance)
- lib/clearlogs.p

# Impacted Systems

BatchOrder table, Goals table, SiteConfigOption (BatchOrder Clear Days), clearlogs.p.

# Traceability

- v-BatchOrder-Clear-Days from site config; purge condition BatchDate < TODAY - v-BatchOrder-Clear-Days
- Do not delete if any Goals reference the batch

# Assertions

- Purge runs for BatchOrder where BatchDate < TODAY - configured clear days.
- Before delete: verify no Goals reference that batch (OrdNum/OrdRefLine or ProdCode/TgtCompDate/BatchDate).
- Default BatchOrder Clear Days = 7.

# Related Information

