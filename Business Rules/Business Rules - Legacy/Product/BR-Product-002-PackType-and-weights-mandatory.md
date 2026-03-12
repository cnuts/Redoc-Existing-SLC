---
Risk-Level: P0
Business-Rule-Id: BR-Product-002
Deprecated: false
---

# Description

PackType must be one of C (Catch), F (Fixed), U (Unipak), K (Key-in), or V (Variance). Schema also allows application support for H (Hybrid) in Set-Label-Wgt. MinWgt, MaxWgt, StdWgt, WgtUnits, WgtRoundOrTruncate, and LabelFile are mandatory.

# Source

- [[../Data Models/Product/Product - legacy]] — Schema (PackType VALEXP C,F,U,K,V), Business Rules Summary
- Set-Label-Wgt: C, U, F, V, H (Hybrid)

# Impacted Systems

Product table, tt-Product, weight validation, Set-Label-Wgt, label and serial creation.

# Traceability

- slc.df Product; "Valid pack types: C=Catch, F=Fixed, U=Unipak, K=Key-in, V=Variance."
- MinWgt 0–9999.98, MaxWgt 0.01–9999.99, StdWgt mandatory

# Assertions

- PackType in (C, F, U, K, V); application may support H.
- MinWgt, MaxWgt, StdWgt mandatory; used for weight validation and label weight calculation.
- MinLabelWgt/MaxLabelWgt used for PackType V (Variance).
- WgtUnits "LB" or "KG" (VALEXP). WgtRoundOrTruncate R or T. LabelFile mandatory.

# Related Information

