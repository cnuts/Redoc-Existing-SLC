---
Risk-Level: P0
Business-Rule-Id: BR-Product-006
Deprecated: false
---

# Description

Label weight is calculated by print/set-label-wgt.p according to PackType. C (Catch): op-LabelWgt = ip-TotalProductWgt. U (Unipak): op-LabelWgt = ip-StdWgt. F (Fixed): op-LabelWgt = min(ip-TotalProductWgt, ip-StdWgt) — if ip-TotalProductWgt LE ip-StdWgt then ip-TotalProductWgt else ip-StdWgt. V (Variance): if ip-TotalProductWgt <= ip-MinLabelWgt then op-LabelWgt = ip-MinLabelWgt; else if ip-TotalProductWgt >= ip-MaxLabelWgt then op-LabelWgt = ip-MaxLabelWgt; else op-LabelWgt = ip-TotalProductWgt. H (Hybrid): op-LabelWgt = ip-StdWgt. Otherwise (any other PackType): op-LabelWgt = ip-TotalProductWgt. Run by Case-print.p, family-case-print.p, case-label.p, case-batch-serial.p.

# Source

- progress-SLC/print/set-label-wgt.p — PROCEDURE Set-Label-Wgt: case ip-PackType when "C" then op-LabelWgt = ip-TotalProductWgt; when "U" then op-LabelWgt = ip-StdWgt; when "F" then op-LabelWgt = (if ip-TotalProductWgt LE ip-StdWgt then ip-TotalProductWgt else ip-StdWgt); when 'V' then clamp to MinLabelWgt/MaxLabelWgt; when "H" then op-LabelWgt = ip-StdWgt; otherwise op-LabelWgt = ip-TotalProductWgt

# Impacted Systems

Product PackType, Serial LabelWgt, print/set-label-wgt.p, case-print, family-case-print, case-label, case-batch-serial.

# Traceability

- [[../Data Models/Product/Product - legacy]] — §7 Label and Set-Label-Wgt
- [[../Drafts/Product/BR-Product-002-PackType-and-weights-mandatory]]

# Assertions

- C: LabelWgt = TotalProductWgt.
- U: LabelWgt = StdWgt.
- F: LabelWgt = min(TotalProductWgt, StdWgt).
- V: LabelWgt = clamp(TotalProductWgt, MinLabelWgt, MaxLabelWgt).
- H: LabelWgt = StdWgt.
- Other: LabelWgt = TotalProductWgt.

# Related Information

