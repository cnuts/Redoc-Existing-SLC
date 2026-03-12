---
Risk-Level: P0
Business-Rule-Id: BR-Locals-007
Deprecated: false
---

# Description

The Locals table is included in system export and import. Export writes Locals to the file locals.d (export.w: when "Local Codes" is selected, export.i with TableName = Locals, ExportFile = locals.d). Import reads from locals.d and uses import2.i with KeyPhrase "Locals.ProductCode = TLocals.ProductCode" for duplicate row matching. The primary key of Locals is (LocalCode, PageNbr); the import key phrase uses only ProductCode, so import match/update behavior is based on ProductCode alone, which may not uniquely identify a row when the same product appears on multiple pages.

# Source

- progress-SLC/system/export.w: WHEN 'Local Codes' THEN {system/export.i &ExportFile = "locals.d" &TableName = "Locals"}
- progress-SLC/system/import.w: TLocals like Locals; WHEN 'Local Codes' THEN {system/import2.i &ImportFile = "locals.d" &TableName = "Locals" &CounterName = "vLocals" &KeyPhrase = "Locals.ProductCode = TLocals.ProductCode"}

# Impacted Systems

Locals table, system/export.w, system/import.w, export.i, import2.i, locals.d file.

# Traceability

- Export: full table dump to locals.d.
- Import: KeyPhrase uses ProductCode only; primary key (LocalCode, PageNbr) is not fully reflected in the import key.

# Assertions

- Counter vLocals tracks number of Locals records imported for the result message.
- Locals is listed in the export/import table selection as "Local Codes".

# Related Information

- [[BR-Locals-001-LocalCode-PageNbr-unique-ProductCode-range]]
