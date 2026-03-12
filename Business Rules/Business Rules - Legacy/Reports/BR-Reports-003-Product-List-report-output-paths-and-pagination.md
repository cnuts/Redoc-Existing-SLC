---
Risk-Level: P1
Business-Rule-Id: BR-Reports-003
Deprecated: false
---

# Description

The Product List report (CreateProductReport in sobjects/s-repviewer.w) writes display output to reports\product.txt and print output to reports\pproduct.txt. The report iterates FOR EACH Product NO-LOCK BREAK BY ProductCode; column headers are localized via get-Lang-Lbl (Reports.ProdList.*). Display output is loaded into the viewer with ediReport:insert-file("reports\product.txt"). When the user clicks Print, ip-printreport regenerates the report to reports\pproduct.txt with paged page-size 66 and then runs adecomm\_osprint.p with "reports\pproduct.txt". So the same Product list is written to product.txt for on-screen viewing (with 17-line page logic in the editor) and to pproduct.txt for printing with 66-line page breaks.

# Source

- progress-SLC/sobjects/s-repviewer.w — CreateProductReport: output stream s-widstream to "reports\product.txt"; for each product no-lock break by productcode; ediReport:insert-file("reports\product.txt"). ip-printreport: output stream s-prtstream to "reports\pproduct.txt" paged page-size 66; for each product no-lock break by productcode; intLineCounter > EDI_PAGE_PRT then page. btnPrint: RUN "adecomm\_osprint.p" (INPUT ?, INPUT "reports\pproduct.txt", ...)

# Impacted Systems

sobjects/s-repviewer.w, Product table, reports\product.txt, reports\pproduct.txt, get-Lang-Lbl (Operator.LangCode), [[BR-Message-001-LangCode-MsgId-unique]].

# Traceability

- [[../Data Models/Reports/Product List Report - legacy]] — Data Source Product table, Output Format, Print Format
- [[BR-Reports-002-Report-viewer-display-and-print-page-sizes]]

# Assertions

- Product List report display file: reports\product.txt; print file: reports\pproduct.txt.
- Print output uses page-size 66; display uses EDI_PAGE_SIZE 17 for paging in the editor.
- Report content is sorted by ProductCode and uses localized column headers.

# Related Information

