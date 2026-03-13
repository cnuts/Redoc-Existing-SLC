# Archive 110 vs Archive 86 — Comparison

Comparison of the two archive MSSQL databases based on live schema queries.

---

## Summary


| Aspect                | Archive 110         | Archive 86     |
| --------------------- | ------------------- | -------------- |
| **Tables**            | 13                  | 12             |
| **Total columns**     | ~434                | ~426           |
| **Extra table**       | ddlevt_log (8 cols) | —              |
| **StatusLog columns** | 16                  | 15             |
| **DB compatibility**  | 150 (SQL 2019)      | 160 (SQL 2022) |
| **SQL Server**        | 2022 RTM-CU20       | 2022 RTM-CU16  |


---

## Table Count


| Archive 110 | Archive 86 |
| ----------- | ---------- |
| 13 tables   | 12 tables  |


**Only in 110:** `ddlevt_log` — DDL/event audit log (EventType, ObjectName, ObjectType, LoginName, UserName, Hostname, TSQLCommand, PostTime).

---

## Column Counts by Table


| Table        | 110    | 86     | Notes                       |
| ------------ | ------ | ------ | --------------------------- |
| ddlevt_log   | 8      | —      | 110 only                    |
| EventLog     | 22     | 22     | Same                        |
| MfgOrd       | 5      | 5      | Same                        |
| OrderDetail  | 62     | 62     | Same                        |
| OrderHeader  | 112    | 112    | Same                        |
| OrderNotes   | 17     | 17     | Same                        |
| OrderSerial  | 7      | 7      | Same                        |
| Rack         | 29     | 29     | Same                        |
| RackLog      | 35     | 35     | Same                        |
| Serial       | 77     | 77     | Same                        |
| SerialLog    | 29     | 29     | Same                        |
| SerialStatus | 8      | 8      | Same                        |
| StatusLog    | **16** | **15** | 86 missing StatusReasonCode |


---

## Schema Differences

### 1. ddlevt_log (110 only)

DDL event log for schema change auditing:


| Column      | Type           |
| ----------- | -------------- |
| EventType   | nvarchar(100)  |
| ObjectName  | nvarchar(128)  |
| ObjectType  | nvarchar(100)  |
| LoginName   | nvarchar(128)  |
| UserName    | nvarchar(128)  |
| Hostname    | nvarchar(128)  |
| TSQLCommand | nvarchar(2000) |
| PostTime    | datetime       |


### 2. StatusLog — Missing Column in 86


| 110 (16 cols)        | 86 (15 cols)   |
| -------------------- | -------------- |
| Operator             | Operator       |
| ChangeDate           | ChangeDate     |
| ChangeTime           | ChangeTime     |
| SerialNum            | SerialNum      |
| Action               | Action         |
| ChangedValue         | ChangedValue   |
| ProcessName          | ProcessName    |
| PreValue             | PreValue       |
| NameValuePairs       | NameValuePairs |
| CodeType             | CodeType       |
| TimeFrame            | TimeFrame      |
| CreateDateTime       | CreateDateTime |
| ModifyDateTime       | ModifyDateTime |
| RecordSeq            | RecordSeq      |
| StatusLogID          | StatusLogID    |
| **StatusReasonCode** | *(missing)*    |


---

## Server / Database Info


|                         | Archive 110                 | Archive 86                  |
| ----------------------- | --------------------------- | --------------------------- |
| **Database name**       | Archive                     | archive                     |
| **SQL Server**          | 2022 RTM-CU20 (16.0.4205.1) | 2022 RTM-CU16 (16.0.4165.4) |
| **Platform**            | Linux (Ubuntu 22.04.5)      | Linux (Ubuntu 22.04.5)      |
| **Compatibility level** | 150 (SQL Server 2019)       | 160 (SQL Server 2022)       |


---

## Interpretation

- **110 vs 86** in the MCP server names likely refer to different sites/plants (e.g. Plant 110, Plant 86), not SQL Server versions.
- **Archive 110** adds:
  - `ddlevt_log` for DDL auditing
  - `StatusReasonCode` in `StatusLog`
- **Archive 86** is the simpler schema; 110 appears to be the more feature-rich variant.

