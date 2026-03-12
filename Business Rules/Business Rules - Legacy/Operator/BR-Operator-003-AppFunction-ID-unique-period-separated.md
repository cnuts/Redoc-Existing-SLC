---
Risk-Level: P0
Business-Rule-Id: BR-Operator-003
Deprecated: false
---

# Description

AppFunction.ID is the primary key of the AppFunction table and must be unique (index ID-index unique, primary, ascending on ID). ID is a period-separated application function identifier (e.g. PrintMenu, ConfigMenu.SiteConfig.Update, Print.Modify.ZuluCapture). The same ID is passed to f-get-permission throughout the app; OperatorAppFunction.ID references AppFunction.ID. New functions are typically added in lib/add-to-runtime-db.p (procedure 1000-AddSecurity).

# Source

- [[../Data Models/Operator/Appfunction-Operator-OperatorAppfunction - legacy]] — §2.2 AppFunction schema
- [[../Data Models/0.Legacy Database Schema]] — AppFunction (ID, Description), index ID-index yes yes ID ASCENDING
- slc.df appfunct

# Impacted Systems

AppFunction table, OperatorAppFunction (ID FK), f-get-permission, all permission-gated UI, add-to-runtime-db.p (1000-AddSecurity).

# Traceability

- AppFunction "Application Functions: menus, programs, buttons, objects"
- ID character x(50), max 100; Description character x(60), max 120
- OperatorAppFunction.ID must equal AppFunction.ID for permission to be meaningful (application-level referential integrity)

# Assertions

- AppFunction.ID is unique (primary key).
- ID is period-separated (e.g. Print.Modify.Price, Setup.Goals).
- OperatorAppFunction.ID references AppFunction.ID; creating OperatorAppFunction rows assumes the ID exists in AppFunction.

# Related Information

