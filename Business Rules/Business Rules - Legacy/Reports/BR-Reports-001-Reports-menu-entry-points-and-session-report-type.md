---
Risk-Level: P1
Business-Rule-Id: BR-Reports-001
Deprecated: false
---

# Description

The Reports menu (sobjects/s-repmenu.w) provides four report entry points. Each button sets the session report context or launches a report window as follows: (1) **List Of Products** — sets session parameter CHR_CURRENTREPORTNAME to "Product" and runs sobjects/s-repviewer.w. (2) **Production Report Interface** — sets session parameter to "Production To Labels" and runs reports/r-prodreps.w. (3) **Item Production Report Interface** — runs reports/itemprod.w (no session report name set in menu). (4) **Serials to CSV** — runs reports/r-SerialsToCSV.w. The report viewer (s-repviewer.w) uses the session parameter to decide which report to generate via StartReport (Product → CreateProductReport, Production → CreateProductionReport).

# Source

- progress-SLC/sobjects/s-repmenu.w — ON CHOOSE btnProductList: f-set-session-parm(CHR_CURRENTREPORTNAME,"Product"), RUN sobjects/s-repviewer.w; bu-PdnReps: f-set-session-parm(...,"Production To Labels"), RUN reports/r-prodreps.w; btnItemProduction: RUN reports/itemprod.w; bu-SerialsToCSV: RUN reports/r-SerialsToCSV.w
- progress-SLC/sobjects/s-repviewer.w — StartReport: chrReportType = f-get-session-parm("CHR", CHR_CURRENTREPORTNAME); case chrReportType: when "Product" run CreateProductReport; when "Production" run CreateProductionReport

# Impacted Systems

sobjects/s-repmenu.w, sobjects/s-repviewer.w, reports/r-prodreps.w, reports/itemprod.w, reports/r-SerialsToCSV.w, session parameter CHR_CURRENTREPORTNAME.

# Traceability

- [[../Data Models/Reports/Production Report - legacy]] — Production report
- [[../Data Models/Reports/Product List Report - legacy]] — Product list
- [[../Data Models/Reports/Serials to CSV Report -legacy]] — Serials to CSV
- [[../Data Models/Reports/Item Production Report - legacy]] — Item Production

# Assertions

- Product List and Production Report Interface set a session report name before opening the viewer or report window; Item Production and Serials to CSV are launched directly without setting that session parm in the menu.
- The report viewer dispatches report generation based on chrReportType from the session parameter.

# Related Information

