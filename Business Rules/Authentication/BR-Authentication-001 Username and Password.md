---
Risk-Level: P0
Business-Rule-Id: BR-Authentication-001
Deprecated: false
---


# Description
A valid user name, password and shift selection is required prior to login attempt
# Source
*   RM8675309 - "Username and password requirements"
# Impacted Systems
all
# Traceability

# Assertions
- User name must be over 8 characters long and only contain lower and uppercase Latin characters.  Maximum length is 15 characters
	- Regular Expression:  ^[a-zA-Z]{9,15}$
- A password of at least 8 characters long and contain only lower and uppercase Latin characters. Maximum length is 20 characters.
	- Regular Expression:  ^[a-zA-Z]{9,20}$
# Related Information


