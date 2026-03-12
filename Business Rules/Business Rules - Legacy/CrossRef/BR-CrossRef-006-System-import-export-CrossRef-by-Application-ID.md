---
Risk-Level: P0
Business-Rule-Id: BR-CrossRef-006
Deprecated: false
---

# Description

System import and export (system/import.w, system/export.w) include the CrossRef table. The import file name is "crossref.d". The key used for matching on import is CrossRef.Application = TCrossRef.Application and CrossRef.ID = TCrossRef.ID. The temp-table tCrossRef is defined like CrossRef. The counter variable vCrossRef is initialized to 'CrossRef = 0' and included in the export summary. Export and import use the same key phrase so that records are updated by (Application, ID) on import.

# Source

- progress-SLC/system/import.w: def temp-table tCrossRef like CrossRef. vCrossRef = 'CrossRef = 0'. &ImportFile = "crossref.d", &TableName = "CrossRef", &CounterName = "vCrossRef". &KeyPhrase = "CrossRef.Application = TCrossRef.Application and CrossRef.ID = TCrossRef.ID".

# Impacted Systems

CrossRef table, system/import.w, system/export.w, crossref.d.

# Traceability

- Key is (Application, ID); consistent with BR-CrossRef-001 and with update-slc crossref.d handling.

# Assertions

- Import matches existing rows by (Application, ID) and updates or creates accordingly per the generic import logic.
- Export writes CrossRef data to crossref.d in the format expected by the import.

# Related Information

- [[BR-CrossRef-001-Application-ID-unique]]
- [[BR-CrossRef-002-update-slc-crossref-d-import-UpdateAvailable-delete]]
