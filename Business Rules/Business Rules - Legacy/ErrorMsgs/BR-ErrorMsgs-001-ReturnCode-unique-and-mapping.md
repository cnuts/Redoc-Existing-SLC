---
Risk-Level: P0
Business-Rule-Id: BR-ErrorMsgs-001
Deprecated: false
---

# Description

ErrorMsgs table holds error codes: ReturnCode (integer) maps to ErrorText (character X(80)). ReturnCode is the primary key and must be unique (index RC unique on ReturnCode ascending). Used for standardized error messages (e.g. host update, update-slc.w).

# Source

- [[../Data Models/0.Legacy Database Schema]] — ErrorMsgs (ReturnCode, ErrorText), index RC yes yes ReturnCode ASCENDING
- [[../Data Models/Message/Message - legacy]] — §1 Overview (ErrorMsgs)
- slc.df errormsg

# Impacted Systems

ErrorMsgs table; host update flows, update-slc.w, any code that looks up error text by return code.

# Traceability

- Description: "Holds Error Codes"
- ReturnCode integer, ErrorText character X(80)

# Assertions

- ReturnCode is unique (primary key).
- Each ReturnCode has one ErrorText; used for standardized error display.

# Related Information

