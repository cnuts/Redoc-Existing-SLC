---
Risk-Level: P0
Business-Rule-Id: BR-UserBar-006
Deprecated: false
---

# Description

For UBField "MANUF_ID", the character is taken from the product-specific MFGID when available. lib/userbar.p finds CrossRef where Application = "Product-MFGID" and ID = string(ProductCode,"99999"). If that row is available and CrossRef.Descr is not "0000000" and not ?, the character is taken from Descr (formatted as "xxxxxxx") at UBFieldCharLoc. Otherwise the character is taken from gMfgID (string(gMfgID,"9999999") at UBFieldCharLoc). So product override is used only when Descr is present and not the sentinel "0000000".

# Source

- progress-SLC/lib/userbar.p — When "MANUF_ID" then do: v-Char = substring(string(gMfgID,"9999999"),…). find MFGID-CrossRef where Application = "Product-MFGID" and ID = string(pb-CaseLabelSource.Product,"99999") no-error. if avail MFGID-CrossRef and MFGID-CrossRef.Descr <> "0000000" and MFGID-CrossRef.Descr <> ? then v-Char = substring(string(MFGID-CrossRef.Descr,"xxxxxxx"), UBDetail.UBFieldCharLoc, 1).

# Impacted Systems

UBDetail (MANUF_ID), CrossRef (Application Product-MFGID), lib/userbar.p, gMfgID (site config).

# Traceability

- [[../Data Models/UserBar/UserBar - legacy]] — §5 MANUF_ID from CrossRef
- [[../Drafts/CrossRef/BR-CrossRef-001-Application-ID-unique]]

# Assertions

- MANUF_ID uses product-specific value from CrossRef (Product-MFGID, ID=ProductCode) only when Descr is not "0000000" and not ?.
- Otherwise MANUF_ID uses gMfgID.

# Related Information

