---
Risk-Level: P0
Business-Rule-Id: BR-Locals-006
Deprecated: false
---

# Description

The page title (pageheader) for a Local Codes page is restricted to 40 characters. In s-localcodes.w the filTitle widget has FORMAT "X(40)" and the Locals.pageheader field is assigned from filTitle on create and on save; the schema defines pageheader as character X(40). Documentation in the source (05/07/01) states that the length of title was restricted to 40 chars.

# Source

- progress-SLC/sobjects/s-localcodes.w: DEFINE VARIABLE filTitle AS CHARACTER FORMAT "X(40)"; ip-updateLocals: Locals.Pageheader = Filtitle / Filtitle:screen-value; comment "05/07/01 ajh - restricted length of title to 40 chars"
- [[../Data Models/0.Legacy Database Schema]] — Locals table, pageheader X(40)

# Impacted Systems

Locals.pageheader, s-localcodes.w (filTitle), display and save of page title.

# Traceability

- UI and database both limit pageheader to 40 characters; no separate truncation logic is required in the save path when using the same format.

# Assertions

- filTitle FORMAT "X(40)" constrains input to 40 characters.
- All Locals rows on the current page share the same Pageheader value after save (see [[BR-Locals-002-ip-updateLocals-create-delete-by-slot-and-pageheader]]).

# Related Information

- [[BR-Locals-001-LocalCode-PageNbr-unique-ProductCode-range]]
