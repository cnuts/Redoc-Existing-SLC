---
Risk-Level: P0
Business-Rule-Id: BR-ProductProcess-010
Deprecated: false
---

# Description

print/getnextprocess.p procedure GetNextProdProcess selects the next process for the current product by finding the first TT-ProductProcess where ProductCode = gTTProductCode and ProcessSequence > iopLastProcessSequence. If no such process exists (all processes for the product have been run), the procedure wraps to the first process: it finds the first TT-ProductProcess where ProductCode = gTTProductCode and ProcessSequence > -1, runs FlushStack in the stack functions handle, sets iopLastProcessSequence to that process's ProcessSequence, clears iopStackGroupID, and sets opNewCase = yes. ProcessID = 'Group' denotes a group header; the procedure uses GroupID and a stack (FlushCurrentGroup, f-process-level) to manage group repetitions and hierarchy (higher/lower/same level). When more repetitions are needed for the current group, it finds the first process with ProcessID = 'Group' and GroupID = iopStackGroupID for that product.

# Source

- progress-SLC/print/getnextprocess.p — FIND first b-tt-ProductProcess where ProductCode = gTTProductCode and ProcessSequence > iopLastProcessSequence. If not avail: FIND first where ProcessSequence > -1, RUN FlushStack, assign iopLastProcessSequence, iopStackGroupID = '', opNewCase = yes. Group logic: f-process-level(b-tt-ProductProcess.GroupID) vs iopStackGroupID; FIND first where ProcessID = 'Group' and GroupID = iopStackGroupID for "start over from beginning of group".

# Impacted Systems

print/getnextprocess.p, TT-ProductProcess, ProcessSequence, ProcessID 'Group', GroupID, stackfunctions (FlushStack, FlushCurrentGroup), f-process-level, NewProductInits, btnAcceptInput.

# Traceability

- [[../Data Models/0.Legacy Database Schema]] — ProductProcess ProcessSequence, ProcessID
- Process flow: local-init via NewProductInits, on choose of btnAcceptInput

# Assertions

- Next process is the first ProductProcess for the product with ProcessSequence > last sequence.
- When no such process exists, wrap to first process (ProcessSequence > -1), FlushStack, set NewCase = yes.
- ProcessID = 'Group' marks group headers; group iteration and repetitions are driven by GroupID and the stack.

# Related Information

