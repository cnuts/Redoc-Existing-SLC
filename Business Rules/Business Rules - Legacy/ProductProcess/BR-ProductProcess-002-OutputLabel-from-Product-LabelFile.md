---
Risk-Level: P0
Business-Rule-Id: BR-ProductProcess-002
Deprecated: false
---

# Description

Product defines LabelFile; the output step's OutputLabel is set from Product.LabelFile when building tt-ProductProcess (weigh-create-tt-prodprocess.p). ProductProcess does not store process sequence in Product; ProductProcess holds ProcessSequence, ProcessID, Routine, Device, Item, etc., per product.

# Source

- [[../Data Models/Product/Product - legacy]] — §4 ProductProcess Link, §7 Label and Set-Label-Wgt
- weigh-create-tt-prodprocess.p; OutputLabel set from b-TT-Product.LabelFile

# Impacted Systems

ProductProcess, Product.LabelFile, tt-ProductProcess.OutputLabel, case-print, PrintLabel.

# Traceability

- ProductProcess.OutputLabel; Product.LabelFile drives label template resolution

# Assertions

- When building tt-ProductProcess for a product, OutputLabel for output steps is set from Product.LabelFile (unless overridden per step).
- Goals.LabelFile can override product label file when producing to goal.

# Related Information

