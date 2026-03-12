---
Risk-Level: P0
Business-Rule-Id: BR-ErrorMsgs-002
Deprecated: false
---

# Description

The include file lib/errMsg.i looks up the ErrorMsgs table by ReturnCode (passed as preprocessor argument &errormsg). If a row is found, the variable vMsg is set to ErrorMsgs.ErrorText, then the placeholders [1], [2], [3], [4] in that text are replaced with the preprocessor arguments &p1, &p2, &p3, &p4 respectively. If no row is found, vMsg is set to 'Unknown'. The message is then displayed via run dialogs/d-tsmsgbox.w with title 'ERROR' and the message vMsg. The calling code must define vMsg and pass &errormsg and optionally &p1 through &p4 when including errMsg.i.

# Source

- progress-SLC/lib/errMsg.i: FIND first ErrorMsgs WHERE ErrorMsgs.ReturnCode = {&errormsg} no-lock no-error; IF AVAIL then vMsg = ErrorMsgs.ErrorText, REPLACE [1]..[4] with {&p1}..{&p4}; ELSE vMsg = 'Unknown'; RUN dialogs/d-tsmsgbox.w (input 'ERROR', input vMsg)
- Comment in file: "display a dialog box with an error msg, replacing the arguments with the passed vars"; format example {lib/errmsg.i &errormsg="33" &p1="value 1" ... &p4="value 4"}

# Impacted Systems

ErrorMsgs table, lib/errMsg.i, dialogs/d-tsmsgbox.w, any procedure that includes errMsg.i with &errormsg and &p1–&p4.

# Traceability

- ErrorText is the single source for the displayed message when the ReturnCode exists; up to four placeholders allow parameterized messages.
- When ReturnCode is missing from the table, the user sees "Unknown" instead of raw code or blank.

# Assertions

- Lookup is by ReturnCode only (primary key).
- Placeholders [1] through [4] are replaced by string substitution; any other bracketed text is not modified by this include.
- d-tsmsgbox is always run with title 'ERROR' and the (possibly replaced) vMsg.

# Related Information

- [[BR-ErrorMsgs-001-ReturnCode-unique-and-mapping]]
