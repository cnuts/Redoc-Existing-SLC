---
Risk-Level: P0
Business-Rule-Id: BR-UserBar-007
Deprecated: false
---

# Description

For UBField "CHECKSUM13" or "CHECKSUM31", UBDetail.UBFieldCharLoc is the start position and UBDetail.UBLength is the length of the substring of the userbar string used as input to the check-digit calculation. During the loop, a placeholder character "E" is written at the current position; after the loop, procedure checksumvalue is run with that substring and weight type (13 or 31). The computed check digit (mod 10, alternating weights 1 and 3) overwrites the "E" in v-userbar at that position.

# Source

- progress-SLC/lib/userbar.p — When "CHECKSUM13" then v-checksum13pos = (v-currentpos + 1), v-char = "E", v-startpos13 = UBFieldCharLoc, v-length13 = UBLength. When "CHECKSUM31" then v-checksum31pos = (v-currentpos + 1), v-char = "E", v-startpos31 = UBFieldCharLoc, v-length31 = UBLength. After loop: substring(v-userbar, v-startpos13, v-length13) and checksumvalue(Input 13) then substring(v-userbar, v-checksum13pos, 1) = string(v-checkdigit). Same for 31. Procedure checksumvalue: v-checkdigit 1 or 3, sum of (digit * weight) mod 10, result = (10 - (v-checksumvalue mod 10)) mod 10.

# Impacted Systems

UBDetail (UBFieldCharLoc, UBLength for CHECKSUM13/31), lib/userbar.p.

# Traceability

- [[../Data Models/UserBar/UserBar - legacy]] — §3 UBLength for checksum, §5 CHECKSUM13/CHECKSUM31
- [[BR-UserBar-002-UBDetail-UBName-UBDispLoc-unique-FK-to-UBHeader|BR-UserBar-002]]

# Assertions

- CHECKSUM13 and CHECKSUM31 use UBFieldCharLoc as start and UBLength as length for the substring passed to checksumvalue.
- The check digit overwrites the placeholder position in the final userbar string.

# Related Information

