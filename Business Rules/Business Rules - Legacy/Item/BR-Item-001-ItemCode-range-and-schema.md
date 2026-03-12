---
Risk-Level: P0
Business-Rule-Id: BR-Item-001
Deprecated: false
---

# Description

ItemCode must be in the range 00001–99999. ItemCode is the primary key of the Item table. Desc1/Desc2 length ≤ 32; ShortDesc1/2/3 length ≤ 8. PackType C|F|U|K|V; MinWgt 0–9999.98; MaxWgt 0.01–9999.99; WgtUnits LB|KG; WgtRoundOrTruncate R|T; ExtraTareType N|W|P. LabelFile, ItemDateFormat, PackType, MinWgt, StdWgt, MaxWgt, WgtUnits, WgtRoundOrTruncate are mandatory.

# Source

- [[../Data Models/Item/Item - legacy]] — Schema, Business Rules Summary (§8.1)
- [[../Data Models/0.Legacy Database Schema]] — Item table, index item

# Impacted Systems

Item table, ItemModify, ItemSerial, ProductProcess.Item, tt-Item.

# Traceability

- slc.df Item table; VALEXP for ItemCode, PackType, WgtUnits, ExtraTareType

# Assertions

- ItemCode 1–99999, unique primary key.
- ItemModify: ItemCode 1–99999; ModText1/ModText2 length ≤ 20.
- ItemSerial: AddOrDel A or D only.

# Related Information

