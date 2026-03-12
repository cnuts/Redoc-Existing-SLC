---
Risk-Level: P0
Business-Rule-Id: BR-Auth-009
Deprecated: false
---

# Description

At startup, main.p runs ip-Check-ScaleID before the main loop. ip-Check-ScaleID finds the first SiteConfigOption (primary DB) where OptionName = "ScaleID" and OptionValue = "". If such a row exists, it runs sobjects/s-initsetup.w(OUTPUT vOK) within a transaction. If vOK is FALSE, it leaves the main block (effectively preventing normal run until ScaleID is set). Thus if ScaleID is not configured (empty string), the user must complete initial setup (s-initsetup.w) before the vanity/login flow can proceed; the application does not allow login when ScaleID is blank.

# Source

- progress-SLC/lib/main.p: RUN ip-Check-ScaleID. PROCEDURE ip-Check-ScaleID: FIND FIRST b-SC NO-LOCK WHERE b-SC.OptionName = "ScaleID" AND b-SC.OptionValue = "" NO-ERROR. IF AVAILABLE(b-SC) THEN DO TRANSACTION: RUN sobjects/s-initsetup.w(OUTPUT vOK). IF vOK = FALSE THEN ...
- main loop runs s-vanity.w (logo) which invokes login; ip-Check-ScaleID runs before the loop.

# Impacted Systems

lib/main.p, SiteConfigOption (ScaleID), sobjects/s-initsetup.w, startup sequence.

# Traceability

- ScaleID is required for scale-specific operation; empty ScaleID forces initial setup.

# Assertions

- When ScaleID OptionValue is empty, s-initsetup.w is run; the application does not proceed to vanity/login until setup returns vOK = TRUE or ScaleID is no longer empty.

# Related Information

- [[BR-Auth-001-Username-and-Password-required]]
