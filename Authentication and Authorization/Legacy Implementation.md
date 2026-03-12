# Authentication and Authorization in the SLC Application

This document describes in detail how authentication (login) and authorization (permissions) work in the SLC (Scale Labeling Station) application.

---

## 1. Overview

- **Authentication**: The user identifies themselves with an **Operator ID** and **password**. Validation can be **local** (Operator table + optional MD5 hash) or **remote** (login service over HTTP/HTTPS or socket). Shift and optional department are also captured at login.
- **Authorization**: After login, access to menus and functions is controlled by **application functions** (AppFunction) and **operator permissions** (OperatorAppFunction). A central function **f-get-permission** checks whether the current operator may perform a given function. Special operator **CFSUSER** and the **gSecurityOverride** setting bypass all permission checks.

---

## 2. Database Schema

### 2.1 Operator Table

**Source**: `slc.df`

| Field     | Type     | Description |
|----------|----------|-------------|
| Operator | character | Operator ID (primary key). |
| Password | character | Stored password: either plain text or 32-character hex-encoded MD5 hash. |
| FirstName | character | First name. |
| LastName | character | Last name. |
| LangCode | character | Language code (e.g. English). |

**Index**: `Oper` on `Operator` (ascending).

**Notes**:
- If **Password** length is 32, the application treats it as MD5 (hex). Otherwise it is compared as plain text.
- Operators used as **groups** (e.g. for Active Directory) have **Password = ""** and are not used for local password login; they define permissions that can be applied to a user authenticated by the login service.

### 2.2 AppFunction Table

**Source**: `slc.df`

| Field       | Type     | Description |
|------------|----------|-------------|
| ID         | character | Unique application function identifier (e.g. `PrintMenu`, `ConfigMenu.SetNewScaleConfigs`). |
| Description | character | Human-readable description. |

**Index**: `ID-index` on `ID`.

**Purpose**: Defines the set of permission “gates” in the application. Each gate is one **AppFunction.ID**. The list is seeded and extended in **lib/add-to-runtime-db.p** (see §6).

### 2.3 OperatorAppFunction Table

**Source**: `slc.df`

| Field     | Type    | Description |
|----------|---------|-------------|
| Operator | character | Operator ID (from Operator table). |
| ID       | character | AppFunction ID. |
| Permitted | logical | Whether this operator may perform this function. |

**Indexes**: `OperatorID`, `ID-Operator` (both include Operator and ID).

**Purpose**: Many-to-many link between operators and application functions. A row with **Permitted = true** grants that operator access to that function. **f-get-permission** uses this table keyed by **gOperator** and the requested **AppFunction.ID**.

---

## 3. Authentication: Login Flow

### 3.1 Entry Point and UI

- **File**: `sobjects/s-login.w`
- **Widgets**: Operator ID (**fi_operator**), Password (**fi_PassWd**), optional Confirm Password (**fi_PassWdConfirm**), Shift (**fillshift**), Plant ID (display-only **fi_PlantId**), optional Department (**cboDept**).
- Plant ID comes from site config and is not entered by the user. Shift may be fixed by site config (**SetShiftOnLogin = false**) or must be entered (**SetShiftOnLogin = true**). Valid shifts are defined in **ConfigOptionMaster** under option name **"VALIDSHIFTS"** (acceptable values, pipe-separated).

### 3.2 Main Login Path: ipTryLogin

When the user submits login (e.g. Touch Screen Login button or F2):

1. **Shift** is set from the session or from the widget depending on **SetShiftOnLogin**.
2. **Operator ID** and **Password** are taken from the widgets.
3. If a **login service** is configured (host type, URL/port), the app sends a JSON message with `user`, `pwd`, `type` = `'slc'`, and optionally `continueLogin` to the service (HTTP/HTTPS or socket).
4. **If the service is unreachable**: The app falls back to **ip-AuthLocally** (see §3.4).
5. **If the service responds**:
   - Success/failure and optional error code/message are read from the response.
   - **Password expired** flow: If the server indicates password expired, the UI can show Reset Password; the user enters new password and confirm; the app sends a second request with old and new password and `continueLogin`. On success, login proceeds.
   - **User/Group**: The server can return a **Group** (e.g. Active Directory group). If the operator is not in the local **Operator** table but a **Group** is returned:
     - The app looks up an **Operator** with **Operator = vUserGroup** and **Password = ""** (group definition). If found, it may **CreateTempUser** and use that group’s permissions for the session. If not found, login fails with a message to send the Operator/Group down to the SLC.
     - If the operator exists locally and a group is returned, the app finds the group operator (Password = ""). It then **copies** that group’s **OperatorAppFunction** rows into the current operator’s permissions for the session (creating **OperatorAppFunction** rows for the logging-in operator from the group’s rows). So **group permissions override or supplement** the local operator’s permissions (per comments: group permissions replace local user permissions; if the group has no permissions, the user cannot login).
   - On success, **SetGoodLogin** is run (see §3.5).
6. **If no login service or local fallback**: **ip-AuthLocally** is used with the entered operator and password.

### 3.3 ip-validate-user (Local Validation Only)

Used when the app validates **without** calling the login service (e.g. local-only path or after service unreachable and fallback).

1. **Shift**: Must be in **ConfigOptionMaster "VALIDSHIFTS"** (pipe-separated). Otherwise an error is shown and login stops.
2. **Operator**: **FIND Operator** by **fi_operator**. If not found, show “No Operator” and stop.
3. **Password**:
   - If **LENGTH(Operator.Password) <> 32**: compare **fi_PassWd** to **Operator.Password** as plain text.
   - If **LENGTH(Operator.Password) = 32**: compute **vPWEncoded = HEX-ENCODE(MESSAGE-DIGEST('MD5', fi_PassWd))** and compare to **Operator.Password**. Mismatch → “Bad Password”, clear password field, stop.
4. On success: **SetGoodLogin(output po-ok)**.

### 3.4 ip-AuthLocally

**Purpose**: Validate operator and password using only the local **Operator** table (no login service).

1. **FIND Operator** by **ip-Operator**. If not found, show “No Operator” and return **op-OK = false**.
2. **Password**:
   - If **LENGTH(Operator.Password) <> 32**: compare **ip-Password** to **Operator.Password** as plain text.
   - If **LENGTH(Operator.Password) = 32**: **vPWEncoded = HEX-ENCODE(MESSAGE-DIGEST('MD5', ip-Password))**; compare **vPWEncoded** to **Operator.Password**.
3. On mismatch: message “Bad Password”, clear **fi_PassWd**, return **op-OK = false**.
4. On success: **RUN SetGoodLogin(OUTPUT pook)**.

So local auth supports both **plain text** and **32-char hex MD5** stored passwords.

### 3.5 SetGoodLogin

- Sets **po-OK = true**, **gOperator = fi_operator**, **gShift = INT(fillShift)**.
- Writes a line to **logs/logins.log**: date, time, operator, shift, first name, last name.
- After this, the caller typically runs **assign-login-globals** so the rest of the app sees the correct session globals.

### 3.6 assign-login-globals

- **FIND Operator** where **Operator.Operator = gOperator** (no-lock).
- Runs **{lib/GlobalAssigns.i}** so all session globals (plant, shift, operator, session ID, config options, etc.) are set from site config and operator.
- Sets **gLangCode** from **Operator.LangCode** (default `'English'`).

### 3.7 CFSUSER Backdoor (ip-Validate-Override)

When **gSecurityOverride** is true (from .ini section **Chickway**, key **SecurityOverride** = **yes**), the login path can use **ip-Validate-Override** instead of full password check:

- Shift is still validated against **VALIDSHIFTS**; if missing or invalid, shift is defaulted to **'1'** and a warning is shown but login continues.
- **Operator** is taken from **fi_operator**. If that **Operator** does not exist, a new **Operator** row is **created** with Operator = fi_Operator, FirstName/LastName = fi_Operator, LangCode = 'English'. No password is set.
- **SetGoodLogin** is run.

So with SecurityOverride, the user can log in as any ID (or create an on-the-fly operator) without a password. This is a backdoor for support/admin.

**CFSUSER special operator**:

- **vBackdoorOperator** = **'CFSUSER'**.
- If the user enters operator **CFSUSER** and a **special password** (current time in **HHMM** format, e.g. **vTime = SUBSTRING(string(time,'HH:MM'),1,2) + SUBSTRING(string(time,'HH:MM'),4,2)**), and **v-DisableCFSUserBackdoor** is not set, the code skips normal password check and:
  - Finds or **creates** an **Operator** row for **CFSUSER** (FirstName/LastName/LangCode set to CFSUSER / English).
  - Runs **SetGoodLogin**. So CFSUSER can log in with the current time as password (e.g. 1435 at 2:35 PM).
- **v-DisableCFSUserBackdoor** is set when .ini has **SecurityOverride=DisableCFSUser**, which disables this CFSUSER backdoor (documented for sites that had the algorithm discovered).

### 3.8 Security Override and .ini

- **GET-KEY-VALUE SECTION 'Chickway' KEY 'SecurityOverride'**:
  - **yes** → **gSecurityOverride = yes** (and override login path can be used; also bypasses permission checks in **f-get-permission**).
  - **DisableCFSUserBackdoor** → **v-DisableCFSUserBackdoor = yes** (CFSUSER time password disabled).

---

## 4. Authorization: f-get-permission

**File**: `lib/services.p`

```progress
FUNCTION f-get-permission RETURNS LOGICAL (INPUT pi-AppFunction AS CHAR):

  if gOperator = 'CFSUSER' then
      return true.

  if gSecurityOverride then
      return true.

  find first OperatorAppFunction
      where OperatorAppFunction.ID          = pi-AppFunction
        and OperatorAppFunction.Operator   = gOperator
      no-lock no-error.

  if not avail OperatorAppFunction or not OperatorAppFunction.Permitted then
      return false.
  else
      return true.

END FUNCTION.
```

**Behavior**:

1. If **gOperator** is **CFSUSER** → **true** (full access).
2. If **gSecurityOverride** is true → **true** (full access).
3. Otherwise: look up **OperatorAppFunction** by **ID** = **pi-AppFunction** and **Operator** = **gOperator**. If no row or **Permitted** is false → **false**; else **true**.

So every guarded action in the app calls **f-get-permission** with a string constant (e.g. `"PrintMenu"`, `"ConfigMenu.SetNewScaleConfigs"`). The UI disables or hides buttons/menus when **f-get-permission** returns false.

---

## 5. Application Functions (Seeded and Used)

**File**: `lib/add-to-runtime-db.p` defines parallel arrays **v-AppFunctionID**, **v-AppFunctionDesc**, **v-AppFunctionPermitted**.

**Procedure 1000-AddSecurity** runs at startup (from **add-to-runtime-db.p**). For each non-blank **v-AppFunctionID**:

1. If **AppFunction** with that **ID** does not exist, **CREATE** it and set **Description** from **v-AppFunctionDesc**.
2. For **every Operator** in the database, if **OperatorAppFunction** for that operator and that **AppFunction.ID** does not exist, **CREATE** it and set **Permitted** from **v-AppFunctionPermitted**.

So new functions added to the arrays get a new **AppFunction** row and a default **OperatorAppFunction** row for every existing operator.

**Example seeded functions** (from add-to-runtime-db.p):

| ID | Description | Default Permitted |
|----|-------------|--------------------|
| PrintMenu | Main Menu Print Function | YES |
| ProcModeMenu | Change Processing Mode | NO |
| SystemMenu.LabelEngine | Non-progress function for Label Maint | YES |
| SystemMenu.GraphicsDownload | DataMax function for Image Download | YES |
| Print.Modify.ZuluCapture | Zulu Dept Set on SLC | YES |
| ConfigMenu.SetNewScaleConfigs | Change Main Scale Configs | NO |
| Print.WgtCheckOverride | Wgt Check Override | NO |
| LocalCodes.Product | Select Product | YES |
| Print.OverrideWeight | Override Weight | NO |

These IDs are the same strings passed to **f-get-permission** throughout the app (e.g. menus, print overrides, config screens).

---

## 6. Operator Security UI

**File**: `sobjects/s-operatorsecurity.w`

- Provides a UI to manage **OperatorAppFunction** for operators: view, add, and change **Permitted** for each **AppFunction** per operator.
- Uses **OperatorAppFunction** and **AppFunction** (and operator list). Buttons include Update, Copy (permissions from one operator to another), Quit.
- Access to this screen is itself typically guarded by **f-get-permission** so only authorized users can change permissions.

---

## 7. Session Globals (Relevant to Auth)

**File**: `lib/gLoginVars.i`

- **gOperator**: Current operator ID (set at login).
- **gSecurityOverride**: Set from .ini **Chickway.SecurityOverride**; when true, **f-get-permission** always returns true and override login path can be used.

**File**: `lib/GlobalAssigns.i`

- Assigns **gOperator**, **gPlantID**, **gShift**, **gSessionID**, and many other session parameters from **SiteConfigOption** and operator. Run after login via **assign-login-globals**.

---

## 8. Login Service (External Auth)

- When a **login service** URL/port is configured, the app sends a JSON body with at least **user**, **pwd**, **type** (`'slc'`). For password reset, **pwd2**, **continueLogin**, etc. are used.
- The service can return success/failure, **password expired**, and **Group**. The SLC then:
  - Maps **Group** to an **Operator** with **Password = ""** (group definition).
  - Uses or creates **OperatorAppFunction** for the logging-in user based on that group (see §3.2).
- So **authorization** (which functions the user can perform) is still driven by **Operator** and **OperatorAppFunction** on the SLC; the external service only authenticates and optionally supplies a group whose permissions are applied for that session.

---

## 9. Summary

| Concept | Implementation |
|--------|-----------------|
| **Who can log in** | Operator ID must exist in **Operator** (or be created by override/CFSUSER path). Password: local (plain or MD5 hex) or login service. |
| **Password storage** | **Operator.Password**: plain text or 32-char hex MD5. |
| **Shift** | Must be in **ConfigOptionMaster "VALIDSHIFTS"** unless override/default. |
| **Permission check** | **f-get-permission(pi-AppFunction)** → true if CFSUSER or gSecurityOverride or **OperatorAppFunction** (gOperator, pi-AppFunction) exists and **Permitted** = true. |
| **App functions** | **AppFunction** table; seeded and extended in **add-to-runtime-db.p**; default **OperatorAppFunction** for all operators when a new function is added. |
| **Admin** | **s-operatorsecurity.w** to edit **OperatorAppFunction**; access to it should be permission-controlled. |
| **Backdoors** | **CFSUSER** + time-as-password (unless DisableCFSUserBackdoor); **SecurityOverride=yes** allows override login and full permission bypass. |

---

## 10. Redevelopment Notes

- Replicate **Operator**, **AppFunction**, **OperatorAppFunction** schema and the rule: **Permitted** = true for (Operator, AppFunction.ID) ⇒ allowed.
- Implement **f-get-permission(applicationFunctionId)** with CFSUSER and security-override bypass, then lookup in OperatorAppFunction.
- Support both plain and MD5-hex password storage and comparison during local auth; keep login service contract (JSON in/out, group handling) if external auth is used.
- Seed **AppFunction** and default **OperatorAppFunction** on install/upgrade from a list equivalent to **v-AppFunctionID** / **v-AppFunctionDesc** / **v-AppFunctionPermitted**.
- Preserve **VALIDSHIFTS**, **SetShiftOnLogin**, and **assign-login-globals** behavior so shift and session globals are set consistently after login.
- If CFSUSER or SecurityOverride are retained, implement the same .ini keys and behavior (and document security implications).
