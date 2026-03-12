---
Risk-Level: P0
Business-Rule-Id: BR-UserBar-001
Deprecated: false
---

# Description

UBHeader.UBName is the primary key and must be unique (index UBHIdx). UBName is a single letter A–Z that identifies the userbar; it must match the 8th character of the label token (e.g. USERBARA → "A"). The application supports up to 26 named userbars (USERBARA through USERBARZ). UBLength defines the total length of the userbar string; UBDetail rows should have one row per position (UBDispLoc 1 to UBLength). UBHeader is the master record for one userbar definition.

# Source

- [[../Data Models/UserBar/UserBar - legacy]] — §1 Overview, §2 UBHeader Table
- [[../Data Models/0.Legacy Database Schema]] — UBHeader (UBName, UBLength, UBAddDate, UBAddOper), index UBHIdx yes yes UBName
- slc.df ubheader

# Impacted Systems

UBHeader table, UBDetail (UBName FK), printlabel.p (USERBARA–Z tokens), lib/userbar.p (ip-labelname = SUBSTRING(chrStringToMatch, 8, 1)).

# Traceability

- userbar.p: FIND UBHeader WHERE UBHeader.UBName = ip-labelname; if not available return blank string
- UBName passed from printlabel as single character A–Z

# Assertions

- UBName is unique (primary key).
- UBName is a single letter A–Z corresponding to tokens USERBARA through USERBARZ.
- UBLength = number of UBDetail positions (UBDispLoc 1 to UBLength).

# Related Information

