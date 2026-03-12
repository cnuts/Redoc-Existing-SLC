---
Risk-Level: P0
Business-Rule-Id: BR-Item-006
Deprecated: false
---

# Description

The Find Item dialog (d-itemfind.w) looks up an Item by ItemCode and sets the global gItemRowid. The user enters an item code (fiItemCode). On OK, the procedure finds the first Item where Item.ItemCode >= fiItemCode (GTE lookup). If a row is found and its ItemCode is not equal to the entered value, the message "item &lt;code&gt; Not Found, Found Next" is shown. If no row is found, it finds the last Item; if one exists it shows "Not Found, Found Previous", otherwise "No items Exist". In all cases where an Item row is available after the logic, gItemRowid is set to rowid(Item) and poCancel = false. The dialog is run by sobjects/s-item.w.

# Source

- progress-SLC/dialogs/d-itemfind.w: INPUT ipItemCode, OUTPUT poCancel; assign fiItemCode; FIND first Item WHERE Item.ItemCode >= fiItemCode no-lock no-error; IF AVAIL AND Item.ItemCode <> fiItemCode then d-tsmsgbox "Find item" "item " + string(fiItemCode) + " Not Found, Found Next"; IF NOT AVAIL then FIND last Item; if avail "Not Found, Found Previous" else "No items Exist"; gItemRowid = rowid(Item)

# Impacted Systems

Item table, d-itemfind.w, gItemRowid (gProcessingVars or similar), sobjects/s-item.w, d-tsmsgbox.

# Traceability

- Lookup is greater-than-or-equal; exact match is not required for success but user is informed when only a "next" row is found.
- gItemRowid is used by the caller to position the Item browser or form.

# Assertions

- When no Item exists at all, the message "No items Exist" is shown; caller should verify Item availability when using gItemRowid.
- poCancel is set to false when the find logic completes (OK path); Cancel button would set poCancel true in a separate code path if present.

# Related Information

- [[BR-Item-001-ItemCode-range-and-schema]]
