---
Risk-Level: P1
Business-Rule-Id: BR-Reports-002
Deprecated: false
---

# Description

The report viewer (sobjects/s-repviewer.w) uses fixed page sizes for display and printing. Display page size is 17 lines (EDI_PAGE_SIZE); print page size is 66 lines (EDI_PAGE_PRT). Pagination buttons (Top, Up, Down, Bottom) are shown only when the report has more than (EDI_PAGE_SIZE - 3) lines (i.e. more than 14 lines). When the report has 14 lines or fewer, the page navigation buttons are hidden and disabled. The editor widget displays the report; printing writes to a file with page breaks every 66 lines for printer consumption.

# Source

- progress-SLC/sobjects/s-repviewer.w — &GLOBAL-DEFINE EDI_PAGE_SIZE 17, &GLOBAL-DEFINE EDI_PAGE_PRT 66; IF ediReport:num-lines LE ({&EDI_PAGE_SIZE} - 3) THEN assign btnPageTop:VISIBLE = FALSE, ... else assign btnPageTop:VISIBLE = TRUE, ...; ip-printreport: output stream s-prtstream to "reports\pproduct.txt" paged page-size 66

# Impacted Systems

sobjects/s-repviewer.w (ediReport, btnPageTop/Up/Down/Bottom, ip-printreport), reports\pproduct.txt.

# Traceability

- [[BR-Reports-001-Reports-menu-entry-points-and-session-report-type]] — viewer launched from Reports menu
- [[../Data Models/Reports/Product List Report - legacy]] — Report Characteristics (page breaks every 66 lines)

# Assertions

- Display page size = 17 lines; print page size = 66 lines.
- Page navigation buttons are visible only when report length > 14 lines.

# Related Information

