---
Risk-Level: P0
Business-Rule-Id: BR-UserBar-005
Deprecated: false
---

# Description

Each UBDetail.UBField must match one of the supported names in the CASE CAPS(UBField) block in lib/userbar.p. Supported names include ShiftProdCodeCount, PACK_TYPE, ADJUST, Tare_Number, Wt_Unit_Num, BOXTARE, ExtendedPrice, CUSTOMER, KILLDATE, LABEL_WEIGHT, LONG_DESC1/2, LOT, MANUF_ID, MAX_WEIGHT, MIN_WEIGHT, NET_WEIGHT, ORDER, PACK_DATE, PACK_TIME, PLANT, PRICE, PRODUCT, SCALE, SERIAL, SHIFT, SHORT_DESC1/2/3, STD_WEIGHT, TARE_WEIGHT, TEXT01–TEXT10, VOFFSET, UNIQUE, WT_UNITS, XWT_UNITS, XLABEL_WEIGHT, LITERAL, ASCII, TimeLot, CHECKSUM13, CHECKSUM31, DateAI, DateAIDate. If UBField does not match any WHEN clause, v-Char is not assigned in that iteration; it remains the default (blank), so that position contributes a blank character to the userbar string.

# Source

- progress-SLC/lib/userbar.p — FOR EACH UBDetail … case CAPS(UBField): WHEN "ShiftProdCodeCount" … When "LITERAL" … When "CHECKSUM31" … End Case. v-userbar = v-userbar + v-char. (no OTHERWISE branch)

# Impacted Systems

UBDetail.UBField, lib/userbar.p, label output. UBExamples documents valid UBField names for setup; runtime behavior is defined by userbar.p.

# Traceability

- [[../Data Models/UserBar/UserBar - legacy]] — §3 UBDetail, §5 userbar.p
- [[BR-UserBar-002-UBDetail-UBName-UBDispLoc-unique-FK-to-UBHeader|BR-UserBar-002]]
- [[BR-UserBar-003-UBExamples-UBField-unique|BR-UserBar-003]]

# Assertions

- UBField must be a supported name in userbar.p CASE for that position to contribute the intended character.
- Unsupported UBField results in a blank character at that UBDispLoc.

# Related Information

