---
Risk-Level: P0
Business-Rule-Id: BR-Locals-003
Deprecated: false
---

# Description

The Local Codes screen (s-localcodes.w) uses a page range of 1 to 100 (MIN_PAGE 1, MAX_PAGE 100). Navigating Next from page 100 wraps to page 1; navigating Prev from page 1 wraps to page 100. Each page has 15 button slots (BTN_COUNT 15). The LocalCode value for a slot is derived from the page and button index: v-LocalCodes[slot] = v-Locals + slot where v-Locals is based on the current page (e.g. (page - 1) * 100), so that when loading from the database, Locals.LocalCode MODULO 100 gives the button index (1–15). This maps (LocalCode, PageNbr) to the 15 slots per page.

# Source

- progress-SLC/sobjects/s-localcodes.w: MIN_PAGE 1, MAX_PAGE 100, BTN_COUNT 15; ip-SetCurrentPage: Next at MAX_PAGE → MIN_PAGE, Prev at MIN_PAGE → MAX_PAGE. v-LocalCodes[v-i] = v-Locals + v-i (setup); when loading, v-j = Locals.LocalCode MODULO 100, CASE v-j 1–15 maps to filLocal1Product..filLocal15Product and v-LocalProducts[v-j]
- Comment: "LocalCode array is a concatenation of the page number - 1 and the button number"

# Impacted Systems

s-localcodes.w, Locals table (LocalCode, PageNbr), v-CurrentLocalPage, v-LocalCodes, v-LocalProducts.

# Traceability

- Page numbers 1–100; 15 slots per page; LocalCode encodes both so that (LocalCode, PageNbr) is unique and maps to a single slot.
- Schema LocalCode format "9999", PageNbr format "999"; (99*100+15)=9915 fits.

# Assertions

- v-CurrentLocalPage is constrained to 1–100 with wrap.
- Button index from DB: Locals.LocalCode MODULO 100 yields 1–15 for the slot on that page.

# Related Information

- [[BR-Locals-001-LocalCode-PageNbr-unique-ProductCode-range]]
- [[BR-Locals-002-ip-updateLocals-create-delete-by-slot-and-pageheader]]
