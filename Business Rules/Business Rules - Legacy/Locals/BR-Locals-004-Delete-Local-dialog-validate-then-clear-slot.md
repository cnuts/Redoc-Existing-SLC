---
Risk-Level: P0
Business-Rule-Id: BR-Locals-004
Deprecated: false
---

# Description

The Delete Product Code From Local Codes dialog (d-deleteLocal.w) confirms removal of a product from a local code page. It receives the current page number (pi-PageNbr) and returns the product code to remove (po-ProductCode) and a flag (po-Delete). The user enters a product code (filProductCode). On Yes: the procedure finds a Locals row where ProductCode = filProductCode and PageNbr = pi-PageNbr; if none exists it shows the message PrintLabels.DelLocal.DelTitle / DelMsg and returns without setting po-Delete. If a row exists it sets po-ProductCode = filProductCode and po-Delete = yes and closes. The caller (s-localcodes.w) then clears the corresponding button slot (v-LocalProducts[buttonIndex] = 0); the actual DELETE of the Locals row occurs when the user saves (ip-updateLocals) and the slot is 0.

# Source

- progress-SLC/dialogs/d-deleteLocal.w: INPUT pi-PageNbr; OUTPUT po-ProductCode, po-delete. Btn_Yes: FIND first Locals WHERE Locals.ProductCode = filProductCode AND Locals.PageNbr = pi-PageNbr EXCLUSIVE-LOCK no-error; IF NOT AVAIL(Locals) THEN d-tsmsgbox v-msg-title (PrintLabels.DelLocal.DelTitle), v-msg-txt (PrintLabels.DelLocal.DelMsg), RETURN; ELSE po-ProductCode = filProductCode, po-Delete = yes, CLOSE
- progress-SLC/sobjects/s-localcodes.w: RUN d-deleteLocal (v-CurrentLocalPage, output v-CurrentLocalCode, output vDelete); IF NOT vDelete RETURN; find button index where v-LocalProducts[buttonIndex] = v-CurrentLocalCode; v-LocalProducts[buttonIndex] = 0; RUN ip-ShowNewLocalCode. Actual DELETE in ip-updateLocals when slot = 0

# Impacted Systems

d-deleteLocal.w, s-localcodes.w, Locals table, PrintLabels.DelLocal messages.

# Traceability

- Dialog only validates presence of (ProductCode, PageNbr) and returns the product code; caller clears the slot; persistence of delete is on next Save.

# Assertions

- If no Locals row exists for the entered ProductCode and page, po-Delete is not set to yes and the dialog returns without closing (user can correct or choose No).
- The Locals row is not deleted inside d-deleteLocal; it is deleted in ip-updateLocals when the slot’s v-LocalProducts is 0.

# Related Information

- [[BR-Locals-001-LocalCode-PageNbr-unique-ProductCode-range]]
- [[BR-Locals-002-ip-updateLocals-create-delete-by-slot-and-pageheader]]
