---
Risk-Level: P0
Business-Rule-Id: BR-Item-008
Deprecated: false
---

# Description

Item and ItemModify tables are included in system export and import. Item is exported to a file (export.w lists "Item" in table options; export.i with TableName = Item). Import uses import.i for Item and import2.i for ItemModify: Item is imported with KeyPhrase "Item.ItemCode = TItem.ItemCode"; ItemModify is imported from itemmodi.d with KeyPhrase "ItemModify.ItemCode = TItemModify.ItemCode". Counters vItem and vItemModify track the number of records imported for the result message. Thus both tables use ItemCode as the key for matching existing rows during import.

# Source

- progress-SLC/system/export.w: "Item" in LIST-ITEMS; WHEN 'Item' export with TableName Item (and ItemModify in "Item & Modify" or separate)
- progress-SLC/system/import.w: TItem like Item, TItemModify like ItemModify; vItem, vItemModify counters; WHEN 'Item' import Item (import.i); import2.i for Item with KeyPhrase Item.ItemCode = TItem.ItemCode; import2 for ItemModify from itemmodi.d, KeyPhrase ItemModify.ItemCode = TItemModify.ItemCode

# Impacted Systems

Item table, ItemModify table, system/export.w, system/import.w, export.i, import.i, import2.i, item.d, itemmodi.d.

# Traceability

- ItemCode is the primary key for both Item and ItemModify; import key phrase matches that.

# Assertions

- Item and ItemModify export/import are selected via the system export/import table list.
- KeyPhrase for import determines which existing row is updated when re-importing.

# Related Information

- [[BR-Item-001-ItemCode-range-and-schema]]
