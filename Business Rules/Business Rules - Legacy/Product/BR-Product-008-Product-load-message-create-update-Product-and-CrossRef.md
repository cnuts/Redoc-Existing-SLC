---
Risk-Level: P0
Business-Rule-Id: BR-Product-008
Deprecated: false
---

# Description

When the load-products message is processed (host/msg-load-products.p), the message body is read as JSON into temp-table tt-Product (LIKE Product plus NameValuePairs and MfgID). For each tt-Product: (1) Product is found by ProductCode (exclusive-lock); if not available, a new Product is created. The tt-Product is buffer-copied to Product. (2) CrossRef with Application = "Product" and ID = string(ProductCode,"99999") is created if not found; its Descr is set to tt-Product.NameValuePairs. (3) CrossRef with Application = "Product-UCCGTINPackType" and ID = product code is created if not found; Descr is set from NVP "UCCGTINPackType" or "9". (4) For Product-MFGID: if tt-Product.MfgID is an integer in range 1 to 9999999, CrossRef Application "Product-MFGID" and ID = product code is created or updated with Descr = string(MfgID,"9999999"); otherwise if MfgID is null or out of range, that CrossRef is deleted if it exists. (5) If NameValuePairs "TareToUse" is an integer in 0–8, Modify for that Product is found or created and Modify.TareToUse is set.

# Source

- progress-SLC/host/msg-load-products.p — FOR EACH tt-Product: FIND Product by ProductCode; CREATE if not avail; BUFFER-COPY tt-Product TO Product. CrossRef "Product" (Descr = NameValuePairs). CrossRef "Product-UCCGTINPackType" (Descr from NVP or "9"). CrossRef "Product-MFGID": create/update Descr when MfgID 1–9999999, else delete if avail. TareToUse 0–8: FIND/CREATE Modify, set TareToUse.

# Impacted Systems

host/msg-load-products.p, Product table, CrossRef (Application Product, Product-UCCGTINPackType, Product-MFGID), Modify.TareToUse, messaging host.

# Traceability

- [[BR-Product-001-ProductCode-range-unique]] — Product identified by ProductCode
- [[BR-Modify-001-One-per-Product]] — Modify per Product; load message can create Modify for TareToUse

# Assertions

- Load-products message creates or updates Product from JSON; creates/updates Product, Product-UCCGTINPackType, and Product-MFGID CrossRef; and sets Modify.TareToUse when TareToUse is 0–8.
- MfgID for Product-MFGID must be in range 1–9999999 for create/update; otherwise the Product-MFGID CrossRef is removed.

# Related Information

