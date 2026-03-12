---
Risk-Level: P0
Business-Rule-Id: BR-Serial-002
Deprecated: false
---

# Description

Every Serial must have a valid ProductCode (foreign key to Product). The current operator must be captured for audit trail. ScaleNumber and Shift must be captured at creation.

# Source

- [[../Data Models/Serial/Serial - legacy]] — Creation Rules, Data Integrity Rules
- Application logic: createserial.p, Serial table schema

# Impacted Systems

Serial table, createserial.p, Serial creation and Totals/Goals update.

# Traceability

- [[../Data Models/0.Legacy Database Schema]] — Serial (ProductCode, Operator, ScaleNumber, Shift)
- [[../Data Models/Operator/Operator - legacy]] — Operator and Serial records

# Assertions

- ProductCode is required; must exist in Product table.
- Operator (when specified) must exist in Operator table.
- PrintDate and PrintTime are set at creation (PrintDate = TODAY, PrintTime = SECONDS(MTIME)).
- Operator, Shift (1–3), and ScaleNumber are captured for each Serial.

# Related Information

