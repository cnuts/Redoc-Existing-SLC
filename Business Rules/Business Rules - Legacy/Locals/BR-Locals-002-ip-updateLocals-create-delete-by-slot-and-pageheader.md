---
Risk-Level: P0
Business-Rule-Id: BR-Locals-002
Deprecated: false
---

# Description

When saving local codes in s-localcodes.w (procedure ip-updateLocals), for each of the 15 button slots the application finds Locals by LocalCode (v-LocalCodes[slot]) and PageNbr (v-CurrentLocalPage). If the slot has no product assigned (v-LocalProducts[slot] = 0) and a Locals row exists for that LocalCode and page, the row is deleted. If the slot has a product (v-LocalProducts[slot] <> 0) and no Locals row exists, a new Locals row is created with LocalCode, PageNbr, and Pageheader (from filTitle); then ProductCode is assigned. After processing all slots, every Locals row on the current page has its Pageheader updated to the filTitle screen value (same pageheader for the whole page).

# Source

- progress-SLC/sobjects/s-localcodes.w: ip-updateLocals — FOR v-i 1 TO 15: FIND Locals WHERE LocalCode = v-LocalCodes[v-i] AND PageNbr = v-CurrentLocalPage EXCLUSIVE-LOCK; IF v-LocalProducts[v-i] = 0 AND AVAIL(Locals) THEN DELETE Locals; ELSE IF NOT AVAIL(Locals) THEN CREATE Locals ASSIGN LocalCode, PageNbr, Pageheader = filTitle; ASSIGN Locals.ProductCode = v-LocalProducts[v-i]. FOR EACH Locals WHERE PageNbr = v-CurrentLocalPage: ASSIGN Locals.Pageheader = filTitle:screen-value

# Impacted Systems

Locals table, s-localcodes.w, v-LocalCodes and v-LocalProducts arrays, filTitle (pageheader).

# Traceability

- LocalCode per slot is stored in v-LocalCodes (derived from page and button index; see [[BR-Locals-003-Page-range-and-LocalCode-slot-mapping]]).
- Empty slot (ProductCode 0) causes delete; non-empty slot causes create or update of ProductCode and shared Pageheader.

# Assertions

- Exactly one Pageheader value applies to all Locals rows on the current page after save.
- CREATE assigns LocalCode, PageNbr, Pageheader; ProductCode is set in a separate ASSIGN.

# Related Information

- [[BR-Locals-001-LocalCode-PageNbr-unique-ProductCode-range]]
- [[BR-Locals-003-Page-range-and-LocalCode-slot-mapping]]
