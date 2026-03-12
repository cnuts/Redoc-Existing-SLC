---
Risk-Level: P0
Business-Rule-Id: BR-Product-005
Deprecated: false
---

# Description

The label file used for printing must exist at the path resolved by the system (e.g. SEARCH(ipLabelFile) or "labels/" + TRIM(LabelFile) + ".lbl"). The label file name comes from Product.LabelFile, Item.LabelFile, Goals.LabelFile (when producing to goal), or ProductProcess.OutputLabel (set from Product.LabelFile). If the label file is not found, PrintLblInits returns ? and PrintLabel exits without printing; the system logs and shows "Label File Not Found" / "Missing Label OR folder was moved". If piLabelFile is null (e.g. product has no label file), PrintLblInits can return ? and PrintLabel returns without printing.

# Source

- [[../Data Models/Labels/Labels - legacy]] — §2.2 Path Resolution, §2.1 PrintLblInits; §4 "Label file: Must exist at path returned by SEARCH(piLabelFile); otherwise PrintLblInits returns ? and PrintLabel exits without printing and shows 'Label File Not Found'"
- lib/printlabel.p — PrintLblInits; print/set-labels.p (labels/ + TRIM(op-LabelFile) + ".lbl")

# Impacted Systems

PrintLabel (lib/printlabel.p), PrintLblInits, set-labels.p, Product.LabelFile, Goals.LabelFile, ProductProcess.OutputLabel, case-print and reprint flows.

# Traceability

- Product.LabelFile mandatory (BR-Product-002); Goals.LabelFile can override; OutputLabel from Product.LabelFile (BR-ProductProcess-002)
- Caller typically passes path like "labels/demo.lbl" or name that SEARCH can resolve; chrPathNameToOpen = ? causes PrintLabel to return

# Assertions

- Label file must exist at the resolved path for PrintLabel to produce output.
- If file not found: log, show "Label File Not Found" / "Missing Label OR folder was moved", return; PrintLabel exits without printing.
- If piLabelFile is null (e.g. product has no label file), PrintLabel may return without printing.

# Related Information

