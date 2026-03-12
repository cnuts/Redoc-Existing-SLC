# AppFunction, Operator, and OperatorAppFunction: Concept and Business Rules

This document explains the **Operator**, **AppFunction**, and **OperatorAppFunction** entities in full: schema, relationships, how they drive authorization (who can see or use which controls), business rules, maintenance UI, and integration with login and session globals.

---

## 1. Overview

These three entities implement **user identity** and **function-based authorization** in the SLC:

| Entity | Purpose |
|--------|--------|
| **Operator** | Identifies a user (login name, password, name, language). One row per user (or per “group” when using external auth). |
| **AppFunction** | Defines a permission “gate”: a unique ID (e.g. `ConfigMenu.SiteConfig.Update`, `Print.Modify.Price`) that the application checks to show or enable a menu, button, or screen. |
| **OperatorAppFunction** | Links Operator to AppFunction. A row with **Permitted = true** means that operator may perform that function; the UI uses **f-get-permission( AppFunction.ID )** to decide visibility and enablement. |

**Authorization flow:** At login, **gOperator** is set to the current user’s Operator ID. Whenever the app needs to know “can this user do X?”, it calls **f-get-permission( *AppFunction.ID* )**. That function returns **true** only if the user is **CFSUSER**, or **gSecurityOverride** is true, or an **OperatorAppFunction** row exists for (gOperator, that ID) with **Permitted = true**. Otherwise it returns **false**, and the UI disables or hides the control.

---

## 2. Schema (Source: slc.df)

### 2.1 Operator

**Table:** **Operator**  
**Description:** "Contains User info; names, logins, passwords, etc."

| Field | Type | Description / Rules |
|-------|------|---------------------|
| **Operator** | character (X(10), max 20) | **Primary key.** Login name. **MANDATORY.** |
| **Password** | character (X(10), max 20) | Stored password: plain text, or 32-character hex MD5. Empty for “group” operators used with external auth. |
| **FirstName** | character (X(20), max 40) | First name. |
| **LastName** | character (X(20), max 40) | Last name. |
| **LangCode** | character (x(10), max 20) | Language code (e.g. English). Used for **gLangCode** after login. |

**Index:** **Oper** — unique, primary, ascending on **Operator**.

- **Password semantics:** If **LENGTH(Operator.Password) = 32**, the app treats it as MD5 (hex); otherwise it compares plain text. **Password = ""** is used for “group” operators (e.g. Active Directory group); the group’s **OperatorAppFunction** rows are applied to the logging-in user when the login service returns that group.

### 2.2 AppFunction

**Table:** **AppFunction**  
**Description:** "Application Functions: menus, programs, buttons, objects"

| Field | Type | Description / Rules |
|-------|------|---------------------|
| **ID** | character (x(50), max 100) | **Primary key.** Period-separated application function identifier (e.g. `PrintMenu`, `ConfigMenu.SiteConfig.Update`, `Print.Modify.ZuluCapture`). |
| **Description** | character (x(60), max 120) | Human-readable description of the function. |

**Index:** **ID-index** — unique, primary, ascending on **ID**.

- **ID** is the string passed to **f-get-permission** throughout the app. New functions are typically added in **lib/add-to-runtime-db.p** (procedure **1000-AddSecurity**); the same ID can be created in **AppFunction** and default **OperatorAppFunction** rows for all operators.

### 2.3 OperatorAppFunction

**Table:** **OperatorAppFunction**  
**Description:** "Operator : AppFunction is many:many"

| Field | Type | Description / Rules |
|-------|------|---------------------|
| **Operator** | character (x(10), max 20) | Operator ID; references **Operator.Operator**. |
| **ID** | character (x(50), max 100) | AppFunction ID; references **AppFunction.ID**. |
| **Permitted** | logical (yes/no) | Whether this operator is permitted to perform this function. **Initial "no".** |

**Indexes:**

- **OperatorID** — unique, primary: (**Operator** asc, **ID** asc).
- **ID-Operator** — unique: (**ID** asc, **Operator** asc).

So there is at most one row per (Operator, AppFunction.ID). **Permitted = true** grants access; **Permitted = false** or missing row denies it.

---

## 3. Relationships

- **Operator** ↔ **OperatorAppFunction**: One operator has many **OperatorAppFunction** rows (one per AppFunction they have a permission for). **OperatorAppFunction.Operator** references **Operator.Operator**. The schema does not define a foreign key; the relationship is enforced by application logic and by the fact that **f-get-permission** looks up by **gOperator** (which is set from a valid Operator at login).
- **AppFunction** ↔ **OperatorAppFunction**: One AppFunction can have many **OperatorAppFunction** rows (one per operator). **OperatorAppFunction.ID** must equal an **AppFunction.ID** for the permission to be meaningful; again, referential integrity is application-level (e.g. when creating **OperatorAppFunction** from the Operator Security or AppFunction UIs).
- **Operator** is referenced by **Serial.Operator** and **ItemSerial.Operator** (the user who printed the case/item label). Deleting an operator does not cascade to Serial/ItemSerial; those rows can retain the operator ID as historical data.

**Summary:** Operator and AppFunction are independent master tables; **OperatorAppFunction** is the many-to-many link that stores “operator X has permission for function Y (yes/no).”

---

## 4. Authorization: f-get-permission

**Location:** `lib/services.p`, function **f-get-permission**.

**Signature:** `f-get-permission( INPUT pi-AppFunction AS CHAR ) RETURNS LOGICAL`

**Logic:**

1. If **gOperator = 'CFSUSER'** → return **true** (full access; CFSUSER is a special backdoor operator).
2. If **gSecurityOverride = true** → return **true** (full access; set from .ini **Chickway.SecurityOverride = yes**).
3. **FIND FIRST OperatorAppFunction** where **OperatorAppFunction.ID = pi-AppFunction** and **OperatorAppFunction.Operator = gOperator** (no-lock).
4. If **NOT AVAIL OperatorAppFunction** or **NOT OperatorAppFunction.Permitted** → return **false**.
5. Otherwise return **true**.

So a control is allowed only if there is an **OperatorAppFunction** row for the current operator and that function ID, and **Permitted** is true—unless the user is CFSUSER or security override is on.

**Usage in the app:** Call sites pass a literal **AppFunction.ID** string, e.g.:

- `f-get-permission('ConfigMenu.SiteConfig.Update')` — allow editing site config value.
- `f-get-permission('Print.Modify.Price')` — allow modify price on print screen.
- `f-get-permission('SystemMenu.Serials')` — show Serials in system menu.
- `f-get-permission('ConfigMenu.OperatorSecurity.Update')` — show Update/Copy on Operator Security window.

If the result is **false**, the UI typically **disables** or **hides** the button, menu item, or field.

---

## 5. Business Rules

### 5.1 Operator

- **Operator** (the field) is **mandatory** and unique; it is the login name and the key used in **OperatorAppFunction** and in **Serial** / **ItemSerial** for “who printed.”
- **Password**: Stored as plain text or 32-char hex MD5; empty for group operators. Local login validates against **Operator.Password** (plain or MD5 per length); external login service can authenticate and optionally return a **Group** that maps to an Operator with **Password = ""**.
- **Deleting an operator:** The **Operator** viewer (**viewers/v-operator.w**) on delete **deletes all OperatorAppFunction** rows for that operator before deleting the Operator record. **Serial** and **ItemSerial** are not cascade-deleted; they may keep the operator ID for audit.

### 5.2 AppFunction

- **ID** must be unique. It is a free-form string (often period-separated, e.g. `ConfigMenu.EditDevices`). The same string must be used everywhere that function is checked (menus, screens, buttons).
- **Creating a new AppFunction:** In **v-appfunctions.w**, when adding a new **AppFunction** row, the app **creates an OperatorAppFunction** row for **every existing Operator** with **Permitted = no** so that each operator has an explicit (denied) permission for the new function until an admin grants it.
- **Deleting an AppFunction:** Before deleting the **AppFunction** row, the app **deletes all OperatorAppFunction** rows where **OperatorAppFunction.ID = AppFunction.ID** (see **v-appfunctions.w** local-delete-record). So no orphan OperatorAppFunction rows remain.

### 5.3 OperatorAppFunction

- **Unique (Operator, ID):** At most one row per operator per function. **Permitted** on that row is the only source of truth for “can this operator perform this function?” (aside from CFSUSER and gSecurityOverride).
- **No row or Permitted = false:** **f-get-permission** returns false; the corresponding control is disabled or hidden.
- **Permitted = true:** **f-get-permission** returns true (assuming not CFSUSER/gSecurityOverride, which bypass the table).
- **Copy (Operator Security):** The “Copy” button in **s-operatorsecurity.w** copies permissions from one operator to another: it **deletes all OperatorAppFunction** rows for the *target* operator, then **buffer-copy**s each **OperatorAppFunction** row from the *source* operator into new rows with **Operator = target**. So the target’s permissions are replaced by the source’s.

### 5.4 Session and Bypasses

- **gOperator** is set at login to the current user’s **Operator** value. **f-get-permission** always uses **gOperator** when looking up **OperatorAppFunction**.
- **CFSUSER:** If **gOperator = 'CFSUSER'**, **f-get-permission** always returns true. CFSUSER can log in with a special password (e.g. current time as HHMM) when **v-DisableCFSUserBackdoor** is not set.
- **gSecurityOverride:** Set from .ini **Chickway.SecurityOverride = yes**. When true, **f-get-permission** always returns true and override login can skip normal password validation. Used for support/admin backdoors.

---

## 6. Seeding and Adding New Functions

**Procedure:** **1000-AddSecurity** in **lib/add-to-runtime-db.p**.

**Arrays (examples):** **v-AppFunctionID**, **v-AppFunctionDesc**, **v-AppFunctionPermitted** (parallel arrays; up to 100 entries; empty ID stops the loop).

**Logic for each non-blank v-AppFunctionID:**

1. **FIND FIRST AppFunction** where **AppFunction.ID = v-AppFunctionID[ v-Entry ]**. If **NOT AVAIL**:
   - **CREATE AppFunction**, set **ID** and **Description** from the arrays.
   - For **EACH Operator** (no-lock):
     - If **OperatorAppFunction** for (Operator.Operator, this AppFunction.ID) does not exist, **CREATE** it and set **Permitted** = **v-AppFunctionPermitted[ v-Entry ]**.
2. So when a *new* function is added to the arrays and the procedure runs, every existing operator gets a new **OperatorAppFunction** row with the default **Permitted** value from the array.

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

Many more **AppFunction** IDs are used in the app (e.g. **ConfigMenu.SiteConfig.Update**, **Print.Modify.Price**, **SystemMenu.Serials**, **Product.AddChg**); they may be seeded elsewhere or created manually in the AppFunction maintenance UI.

---

## 7. Maintenance UI

### 7.1 Operators (Operator table)

- **sobjects/s-operators.w** — Operator list/window; access often guarded by **f-get-permission('ConfigMenu.Operators.Update')**.
- **viewers/v-operator.w** — Create/update/delete Operator. On **delete**, deletes all **OperatorAppFunction** for that operator, then deletes the Operator.

### 7.2 AppFunctions (AppFunction table)

- **sobjects/s-cfgmenu.w** — Config menu; entry to “App Functions” is guarded by **f-get-permission('ConfigMenu.AppFunctions')**.
- **viewers/v-appfunctions.w** — Create/update/delete AppFunction. On **add** (new record), creates **OperatorAppFunction** for every Operator with **Permitted = no**. On **delete**, deletes all **OperatorAppFunction** where **ID = AppFunction.ID**, then deletes the AppFunction.

### 7.3 Operator Security (OperatorAppFunction)

- **sobjects/s-operatorsecurity.w** — Manage permissions per operator: select operator, view/edit **OperatorAppFunction** rows (Permitted checkboxes). Buttons:
  - **Update** — save current permission changes.
  - **Copy** — copy permissions from a “from” operator to the currently selected “to” operator (replaces to-operator’s rows with from-operator’s).
  - **Quit** — close.
- Access to Update/Copy is guarded by **f-get-permission('ConfigMenu.OperatorSecurity.Update')**; if the user lacks it, **btnUpdate** and **btnCopy** are hidden.

---

## 8. Reference: AppFunction IDs Used in the Application

The following (and others) are passed to **f-get-permission** in the codebase; each corresponds to an **AppFunction.ID** that can be stored in **OperatorAppFunction** with **Permitted = true** to grant access:

- **Menus:** `ConfigMenu`, `ReportsMenu`, `SystemMenu`, `ProcModeMenu`, `PrintMenu`
- **Config:** `ConfigMenu.SiteConfig.Update`, `ConfigMenu.SiteConfig.Internals`, `ConfigMenu.Master.Update`, `ConfigMenu.Master.Internals`, `ConfigMenu.Operators.Update`, `ConfigMenu.OperatorSecurity.Update`, `ConfigMenu.AppFunctions`, `ConfigMenu.Calibrate`, `ConfigMenu.Master`, `ConfigMenu.SiteConfig`, `ConfigMenu.MonitorIndicator`, `ConfigMenu.SetupPrinter`, `ConfigMenu.TouchPanel`, `ConfigMenu.Operators`, `ConfigMenu.EditUserBar`, `ConfigMenu.EditUserDates`, `ConfigMenu.Translation`, `ConfigMenu.EditDevices`, `ConfigMenu.EditItemModify`, `ConfigMenu.EditItems`, `ConfigMenu.EditProductProcess`, `ConfigMenu.EditValidProcesses`, `ConfigMenu.EditGoals`, `ConfigMenu.EditCrossRef`, `ConfigMenu.SetNewScaleConfigs`
- **System menu:** `SystemMenu.SetTime`, `SystemMenu.EditLabels`, `SystemMenu.Totals`, `SystemMenu.Totals.Update`, `SystemMenu.Serials`, `SystemMenu.Serials.Update`, `SystemMenu.Shutdown`, `SystemMenu.Reboot`, `SystemMenu.Explorer`, `SystemMenu.ClearProduction`, `SystemMenu.RunProgram`, `SystemMenu.Import`, `SystemMenu.Export`, `SystemMenu.DeleteSerials`, `SystemMenu.CommSys`, `SystemMenu.EditItemSerial`, `SystemMenu.ReprintCaseLabel`, `SystemMenu.LabelEngine`, `SystemMenu.Rejection`, `SystemMenu.GraphicsDownload`, `SystemMenu.EditLabels.Update`
- **Reports:** `ReportsMenu.Products`, `ReportsMenu.Production`, `ReportsMenu.SerialsToCSV`
- **Print/weigh:** `Print.Modify`, `Print.Setups`, `Print.OverrideWeight`, `Print.Modify.ZuluCapture`, `Print.Modify.Price`, `Print.Modify.Lot`, `Print.Modify.KillDate`, `Print.Modify.PackDate`, `Print.Modify.VarOffset`, `Print.Modify.Tare`, `Print.Modify.Customer`, `Print.Modify.Order`, `Print.Setups.Goal`, `Print.Setups.Product`, `Print.Setups.Modify`, `Print.Setups.Item`, `Print.Setups.ItemModify`, `Print.Setups.ProductProcess`, `Print.Setups.Pallets`
- **Item:** `Item.AddChg`, `Item.Delete`, `Item.CopyFromProduct`, `Print.ItemModify.Price`, `Print.ItemModify.Lot`, etc.
- **Product:** `Product.AddChg`, `Product.Delete`
- **Local codes:** `LocalCodes.Setup`, `LocalCodes.Product`
- **Goals:** `ConfigMenu.EditGoals.Update`
- **UserBar:** `ConfigMenu.EditUserBar.Update`

(Some IDs may be checked with a parent only, e.g. `ConfigMenu`, to show or hide an entire menu.)

---

## 9. Summary Table

| Entity | Holds | Business rules (summary) |
|--------|--------|---------------------------|
| **Operator** | User identity (login, password, name, LangCode) | Operator mandatory, unique. Password plain or MD5; empty for groups. Delete operator → delete all his OperatorAppFunction first. |
| **AppFunction** | Permission gate ID + description | ID unique. New AppFunction → create OperatorAppFunction (Permitted=no) for every Operator. Delete AppFunction → delete all OperatorAppFunction for that ID. |
| **OperatorAppFunction** | (Operator, AppFunction.ID, Permitted) | One row per (Operator, ID). Permitted = true ⇒ f-get-permission returns true (unless CFSUSER/gSecurityOverride). Copy = replace target’s rows with source’s. |

---

