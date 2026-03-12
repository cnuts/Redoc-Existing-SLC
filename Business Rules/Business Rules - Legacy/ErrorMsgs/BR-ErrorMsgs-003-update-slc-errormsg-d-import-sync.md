---
Risk-Level: P0
Business-Rule-Id: BR-ErrorMsgs-003
Deprecated: false
---

# Description

When the host update process (update-slc.w) processes the file "errormsg.d", it creates a tt-ErrorMsgs row from the import stream (IMPORT tt-ErrorMsgs), finds the ErrorMsgs table row by ReturnCode matching tt-ErrorMsgs.ReturnCode under exclusive-lock. If no ErrorMsgs row exists, it creates one. It then buffer-copies tt-ErrorMsgs to the ErrorMsgs table row and releases the record. Thus ErrorMsgs rows are created or updated by ReturnCode from the errormsg.d file during host sync; the dump name for the table is errormsg (slc.df), and the export file name used in update-slc is errormsg.d.

# Source

- progress-SLC/host/update-slc.w: WHEN "errormsg.d" THEN CREATE tt-ErrorMsgs, IMPORT tt-ErrorMsgs; FIND FIRST ErrorMsgs EXCLUSIVE-LOCK WHERE ErrorMsgs.ReturnCode = tt-ErrorMsgs.ReturnCode NO-ERROR; IF NOT AVAILABLE THEN CREATE ErrorMsgs; BUFFER-COPY tt-ErrorMsgs TO ErrorMsgs; RELEASE ErrorMsgs
- tt-ErrorMsgs is defined LIKE ErrorMsgs; EMPTY TEMP-TABLE tt-ErrorMsgs is used when clearing temp tables for the update process.

# Impacted Systems

ErrorMsgs table, host/update-slc.w, tt-ErrorMsgs temp table, errormsg.d import file.

# Traceability

- Sync is keyed only by ReturnCode; no delete of existing ErrorMsgs rows is performed in this block—only create or update.
- Import format must match the ErrorMsgs table structure (ReturnCode, ErrorText) for buffer-copy to apply correctly.

# Assertions

- Matching is by ReturnCode; one row per ReturnCode in errormsg.d.
- Existing ErrorMsgs row with same ReturnCode is overwritten by buffer-copy from tt-ErrorMsgs.

# Related Information

- [[BR-ErrorMsgs-001-ReturnCode-unique-and-mapping]]
