---
Risk-Level: P0
Business-Rule-Id: BR-Auth-00002
Deprecated: false
---

# Description

- A password shall not be stored in the database at less than SHA512.
- The salt must be generated using crypto.randomBytes() method and stored in the clear, associated to the user. Use 32 bytes as the standard for uniqueness.

# Source

- Security policy / redevelopment requirements
- Node.js crypto: https://nodejs.org/api/crypto.html#cryptorandombytessize-callback

# Impacted Systems

Login service, operator credential storage, any system that persists or verifies passwords.

# Traceability

- [[../Authentication and Authorization/Login Service]]
- [[../Business Rules/Authentication/BR-Auth-00002 Password storage must stored with SHA512 or greater and include a unique salt]]

# Assertions

- Password hash algorithm must be SHA512 or stronger.
- Salt must be generated with crypto.randomBytes(); length 32 bytes.
- Salt is stored in clear text, linked to the user record.

# Related Information

https://nodejs.org/api/crypto.html#cryptorandombytessize-callback
