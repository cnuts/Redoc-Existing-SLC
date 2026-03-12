---
Risk-Level: P0
Business-Rule-Id: BR-Labels-003
Deprecated: false
---

# Description

When printing case labels, the choice between the "first" and "quick" label file is based on whether the current device and label match the previous run. In case-label.p (and similar flows), after Set-Labels returns vFirstLabel and vQuickLabel, vLabelFile is set to vQuickLabel if the combined device and label (vDeviceLabelFile) equals iopCurrentLabelFile (the current label file in use); otherwise vLabelFile is set to vFirstLabel. Then iopCurrentLabelFile is updated to vDeviceLabelFile. Thus the first label of a run (or when switching product/device) uses the main .lbl file (FirstLabel); subsequent labels in the same run use the QuickLabel (.1st or .2nd when present) to avoid re-reading the same header content.

# Source

- progress-SLC/print/case-label.p: vDeviceLabelFile = b-tt-ProductProcess.Device + b-tt-ProductProcess.OutputLabel; RUN Set-Labels IN gh_Set-Labels (b-tt-ProductProcess.OutputLabel, OUTPUT vFirstLabel, vQuickLabel, vCancelLabel, vLabelFile); vLabelFile = (IF vDeviceLabelFile = iopCurrentLabelFile THEN vQuickLabel ELSE vFirstLabel); iopCurrentLabelFile = vDeviceLabelFile
- Comment in case-label: "if not first label, use 2ND (may be same as first)"

# Impacted Systems

case-label.p, set-labels.p (op-FirstLabel, op-QuickLabel), iopCurrentLabelFile, device + OutputLabel combination.

# Traceability

- Same device+label as current → use QuickLabel (e.g. .2nd or .1st).
- Different device or label → use FirstLabel (.lbl).

# Assertions

- When .1st/.2nd do not exist, Set-Labels returns op-QuickLabel = op-FirstLabel, so the same file is used in both cases.
- iopCurrentLabelFile is updated after selecting vLabelFile so the next print in the same context uses QuickLabel.

# Related Information

- [[BR-Labels-001-Set-Labels-path-resolution-lbl-1st-2nd-can]]
