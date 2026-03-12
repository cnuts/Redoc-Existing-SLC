---
Risk-Level: P0
Business-Rule-Id: BR-DateCode-003
Deprecated: false
---

# Description

The Create Date Codes dialog (d-createdatecodes.w) creates DateCode rows for a chosen month or for a full year. Create Month requires both month and year to be chosen; if either is missing, the message "You Must Choose Month and Year" is shown and the action returns. Create Year requires the year to be chosen; if missing, "You Must Choose Year" is shown. For each valid date in the range (day 1–31 for the month, or all months 1–12 and days 1–31 for the year), the procedure assigns vDate and creates a DateCode row with DateCode.Date = vDate. If error-status:error occurs (e.g. duplicate Date), vDuplicatesExist is set to yes, the create is undone (undo block), and the loop continues to the next date without creating a null record. After the loop, poCancel is set to false and poRowidDateCode is set to the rowid of the first DateCode with date >= vFirstOfMonth (Create Month) or date >= vFirstOfYear (Create Year). A success message is shown, including "; Duplicates were bypassed" when vDuplicatesExist is true. Invalid dates (e.g. day 31 for February) are skipped via error-status:error on the date assignment and leave the inner loop.

# Source

- progress-SLC/dialogs/d-createdatecodes.w: btnCreateMonth ON CHOOSE: if cbo-month or cbo-year screen-value = ? then d-tsmsgbox "You Must Choose Month and Year", return. vDate = date(month/day/year). DO vDay = 1 TO 31 (Create Month) or DO vMonth 1 TO 12, vDay 1 TO 31 (Create Year): CREATE DateCode, ASSIGN DateCode.Date = vDate no-error; IF error-status:error THEN vDuplicatesExist = yes, UNDO, NEXT. After loop: FIND first DateCode WHERE DateCode.date >= vFirstOfMonth (or vFirstOfYear), poRowidDateCode = rowid(DateCode). d-tsmsgbox with "Duplicates were bypassed" when vDuplicatesExist. btnCreateYear: same pattern, "You Must Choose Year" if year missing.

# Impacted Systems

DateCode table, d-createdatecodes.w, s-datecodes.w (caller).

# Traceability

- Only DateCode.Date is assigned on create; Code is not set (remains initial or is edited elsewhere).
- Duplicate dates cause a unique constraint error; the procedure bypasses and continues.

# Assertions

- Create Month creates at most 31 rows (one per day that is valid for the month).
- Create Year creates up to 366 rows (valid dates only; invalid date assignment leaves the inner loop for that day).
- poRowidDateCode is set to the first DateCode in the created range (>= first of month/year).

# Related Information

- [[BR-DateCode-001-Date-unique]]
