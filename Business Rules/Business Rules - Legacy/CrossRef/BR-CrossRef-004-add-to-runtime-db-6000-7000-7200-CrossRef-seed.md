---
Risk-Level: P0
Business-Rule-Id: BR-CrossRef-004
Deprecated: false
---

# Description

add-to-runtime-db.p seeds or ensures CrossRef rows via three procedures. 6000-AddCrossRef(ip-Application, ip-ID, ip-Descr): finds CrossRef by Application and ID (exclusive-lock); if not available creates CrossRef; then assigns Application, ID, and Descr to the row. Thus it create-or-updates by (Application, ID). Used at startup for entries such as OssidOrdNum (Application "OssidOrdNum", ID "", Descr "") and TrayPrinterPlantID ("TrayPrinterPlantID", "P-40183", "Used For Trays"). 7000-AddCrossRef-WPL: finds CrossRef where Application = "WPL-Line-This-Scale" and ID = ""; if not available gets gScaleID from site value ScaleID (default '99'), creates CrossRef with Application "WPL-Line-This-Scale", ID "", Descr = STRING(gScaleID, '99'). 7200-Build-GTIN-CrossRefs: for each Product, finds CrossRef where Application = "Product-UCCGTINPackType" and ID = STRING(Product.ProductCode, "99999"); if not available creates CrossRef with that Application, ID, and Descr = "9". So 7000 and 7200 create only when missing; 6000 create-or-update (assign runs after find/create).

# Source

- progress-SLC/lib/add-to-runtime-db.p: RUN 6000-AddCrossRef(...). RUN 7000-AddCrossRef-WPL. RUN 7200-Build-GTIN-CrossRefs. 6000-AddCrossRef: FIND CrossRef WHERE Application = ip-Application AND ID = ip-ID; IF NOT AVAIL CREATE; ASSIGN Application, ID, Descr. 7000-AddCrossRef-WPL: FIND "WPL-Line-This-Scale" + ID ""; IF NOT AVAIL then gScaleID from f-Get-Site-Value-TT ScaleID, CREATE CrossRef, Descr = STRING(gScaleID,'99'). 7200-Build-GTIN-CrossRefs: FOR EACH Product, FIND CrossRef Product-UCCGTINPackType + STRING(ProductCode,"99999"); IF NOT AVAIL CREATE, Descr "9".

# Impacted Systems

CrossRef table, add-to-runtime-db.p, startup, Product table (for GTIN build), SiteConfigOption/ScaleID.

# Traceability

- 6000-AddCrossRef is called with specific (Application, ID, Descr) for OssidOrdNum and TrayPrinterPlantID.
- 7200-Build-GTIN-CrossRefs ensures one Product-UCCGTINPackType row per Product with default Descr "9".

# Assertions

- 6000-AddCrossRef updates Application, ID, Descr on the found or newly created row.
- 7000 and 7200 only create when no row exists; they do not modify existing rows.

# Related Information

- [[BR-CrossRef-001-Application-ID-unique]]
