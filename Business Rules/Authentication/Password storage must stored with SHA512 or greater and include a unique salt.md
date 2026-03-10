---
Risk-Level: P0
Business-Rule-Id: BR-Auth-00002
Deprecated: false
---


# Description
- A password shall not be stored in the database at less than SHA512.
- The salt must be generated using crypto.randomBytes() method and stored in the clear, associated to the user.  Use 32 bytes as the standard for uniqueness.


# Source


# Impacted Systems

# Traceability

# Assertions

# Related Information

https://nodejs.org/api/crypto.html#cryptorandombytessize-callback

