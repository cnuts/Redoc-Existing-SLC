---
Risk-Level: P0
Business-Rule-Id: BR-ProductProcess-011
Deprecated: false
---

# Description

When building TT-ProductProcess from the database (print/weigh-create-tt-prodprocess.p), if no ProductProcess records exist for the current product (gTTProductCode), the source product code is taken from site config OptionName 'ModeProdCodeDefault' (f-get-Site-Value-TT), defaulting to '99999' if the value is blank, ? or invalid. ProductProcess records are then read for that source product (v-PP-ProdCode). Each ProductProcess is copied to TT-ProductProcess with ProcessSequence set to 10 * ProductProcess.ProcessSequence and ProductCode set to gTTProductCode. When the process is the default output process (ProcessID = vDefaultOPProcess, e.g. 'CaseOutput'), TT-ProductProcess.OutputLabel is set from b-tt-Product.LabelFile. So the default template (typically product 99999 or ModeProdCodeDefault) provides the process list; CaseOutput's label is overridden from the current product's LabelFile.

# Source

- progress-SLC/print/weigh-create-tt-prodprocess.p — IF NOT AVAIL b-ProductProcess: v-PP-ProdCode = f-get-Site-Value-TT('ModeProdCodeDefault', '', '99999', ...); if blank/?/error then v-PP-ProdCode = 99999. for each ProductProcess where ProductCode = v-PP-ProdCode: create b-tt-ProductProcess, buffer-copy, assign ProcessSequence = 10 * ProductProcess.ProcessSequence, ProductCode = gTTProductCode. if ProcessID = vDefaultOPProcess then b-tt-ProductProcess.OutputLabel = b-tt-Product.LabelFile.

# Impacted Systems

print/weigh-create-tt-prodprocess.p, ProductProcess, TT-ProductProcess, SiteConfigOption (ModeProdCodeDefault), Product.LabelFile, [[BR-ProductProcess-002-OutputLabel-from-Product-LabelFile]].

# Traceability

- [[BR-ProductProcess-002-OutputLabel-from-Product-LabelFile]] — CaseOutput OutputLabel from Product.LabelFile
- ProductCode 99999 (or ModeProdCodeDefault) is the standard template for products with no ProductProcesses

# Assertions

- When no ProductProcess exists for the current product, template is taken from ModeProdCodeDefault (default 99999).
- TT ProcessSequence = 10 * source ProductProcess.ProcessSequence; TT ProductCode = current product.
- For ProcessID = default output process (e.g. CaseOutput), OutputLabel = current product's LabelFile.

# Related Information

