---
Risk-Level: P0
Business-Rule-Id: BR-Goals-013
Deprecated: false
---

# Description

Goals are included in the system export and import. Export writes Goals to a file (goals.d when selected from the export table list). Import uses a temp-table TGoals like Goals and matches existing rows by GoalID: KeyPhrase is "Goals.GoalID = TGoals.GoalID". The counter vGoals tracks the number of Goals records imported for the result message. Import and export are selected via the "Goals" option in the system import/export table list.

# Source

- progress-SLC/system/export.w: "Goals" in LIST-ITEMS; WHEN 'Goals' export with TableName Goals
- progress-SLC/system/import.w: TGoals like Goals; vGoals counter; WHEN 'Goals' import from goals.d, KeyPhrase Goals.GoalID = TGoals.GoalID

# Impacted Systems

Goals table, system/export.w, system/import.w, goals.d, import.i (or equivalent).

# Traceability

- GoalID is the primary key of Goals; import uses it to update or insert.

# Assertions

- Export dumps Goals to the configured export file for the Goals table.
- Import matches on GoalID to update existing or create new Goals rows.

# Related Information

- [[BR-Goals-005-Host-download-and-Counter-Goal]]
