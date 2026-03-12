---
Risk-Level: P0
Business-Rule-Id: BR-Auth-008
Deprecated: false
---

# Description

After successful login, s-login.w runs assign-login-globals. That procedure finds the Operator row where Operator.Operator = gOperator (no-lock), then runs the include lib/GlobalAssigns.i. GlobalAssigns.i assigns many global variables (from gLoginVars.i and elsewhere) from site config and other sources—e.g. gScaleID, gPlantID, gShift, gVersion, and scores of feature flags and limits. If the Operator row is available, assign-login-globals also sets gLangCode = Operator.LangCode; otherwise gLangCode = 'English'. Thus the session's globals are populated from config and the Operator record immediately after login; the same GlobalAssigns.i is used when the operator quits from site config (s-sitecfg.w) so that config changes take effect.

# Source

- progress-SLC/sobjects/s-login.w: PROCEDURE assign-login-globals. FIND first Operator WHERE Operator.Operator = gOperator no-lock. {lib/GlobalAssigns.i}. IF avail Operator THEN gLangCode = Operator.LangCode. ELSE gLangCode = 'English'.
- progress-SLC/lib/globalassigns.i: ASSIGN gScaleID, gPlantID, gShift, gVersion, ... from f-get-session-parm and config.

# Impacted Systems

s-login.w, lib/globalassigns.i, lib/gLoginVars.i, Operator table, SiteConfigOption, session globals.

# Traceability

- GlobalAssigns.i is included in s-login.w and sobjects/s-sitecfg.w; it reads site config and sets gLoginVars used for the rest of the session.

# Assertions

- gOperator must already be set (by SetGoodLogin) before assign-login-globals runs; the FIND uses gOperator to get Operator.LangCode.

# Related Information

- [[BR-Auth-007-SetGoodLogin-gOperator-gShift-log]]
