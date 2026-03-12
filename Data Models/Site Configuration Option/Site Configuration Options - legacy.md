# Site Configuration Concept and Usage in the SLC Application

This document explains the **Site Configuration** concept in full: the two-table model (**ConfigOptionMaster** and **SiteConfigOption**), the in-session cache (**TT-Site-Value**), how options are read and written, when they are loaded, how they drive behavior across the application, and how they relate to other concepts.

---

## 1. Overview

**Site Configuration** is the SLC’s way of storing and retrieving **per-site (per-plant) settings** that control behavior, UI, and integration. There is no single “config file”; instead:

- **ConfigOptionMaster** defines which options exist (name, description, default, acceptable values, whether operators can edit).
- **SiteConfigOption** holds the **current value** for each option at this site (one row per option).
- At runtime, options are read via **f-get-session-parm** (direct from **SiteConfigOption**) or **f-Get-Site-Value-TT** (from an in-memory temp table **TT-Site-Value**), and written via **f-set-session-parm** / **f-Set-Site-Value-TT**, which update both the database and the cache.

Many **global variables** (e.g. **gScaleID**, **gFalseMidnight**, **gModifyCustomer**) are set at startup from site configuration so the rest of the app uses a single, consistent view of “current site settings.”

---

## 2. Schema

### 2.1 ConfigOptionMaster

**Purpose:** Master list of all possible configuration options (metadata and defaults).

**Source:** `slc.df`, table **ConfigOptionMaster** (DESCRIPTION: "This is the master table of all possible config options")

| Field | Type | Description |
|-------|------|-------------|
| **OptionName** | character (x(30)) | **Primary key.** Unique option identifier (e.g. `FalseMidnight`, `ScaleID`, `ValidShifts`). |
| **OptionDescr** | character (x(60)) | Description of what the option controls. |
| **OptionValue** | character (x(60)) | Default or recommended value. |
| **AcceptableValues** | character (X(60)) | Pipe-separated list of valid values (e.g. `YES|NO`, `02:00|22:00|24:00`). Used by UI for dropdowns and validation. |
| **OperatorEdits** | logical | Whether operators are allowed to edit this option (vs. internal/admin only). |

**Index:** Unique primary on **OptionName**.

- Options are often created or ensured by **lib/add-to-runtime-db.p** (e.g. Zulu, False Midnight, Lot, Processing Mode) or by **Create-SiteConfig** when a value is first requested and the option does not exist.

### 2.2 SiteConfigOption

**Purpose:** Per-site runtime value for each option (one row per OptionName).

**Source:** `slc.df`, table **SiteConfigOption** (DESCRIPTION: "This table contains the configuration option records implemented at this site/plant")

| Field | Type | Description |
|-------|------|-------------|
| **OptionName** | character (x(30)) | **Primary key.** Must match an option defined (or definable) in ConfigOptionMaster. |
| **OptionValue** | character (x(60)) | Current value for this site. |

**Index:** Unique primary on **OptionName**.

- If code requests an option that does not exist in **SiteConfigOption**, the system can **auto-create** a row (and optionally a **ConfigOptionMaster** row) via **lib/create-sitecfg.p**, using a caller-supplied default and description.

---

## 3. Runtime Model: TT-Site-Value and Access Functions

### 3.1 Temp Table TT-Site-Value

**Defined in:** `lib/services.p`

- **TT-Site-Value** is a **temp table** with **OptionName** and **OptionValue**.
- It is populated when **services.p** is first run by **Load-TT-Site-Values**, which:
  - Iterates over all **SiteConfigOption** rows (in the primary DB, e.g. **slc**) and creates a corresponding **TT-Site-Value** row for each.
  - Adds a few non-DB entries (e.g. **SecurityOverride**, **Palletize**) from the Windows registry (Chickway section).

So **TT-Site-Value** is an **in-memory copy** of site config (plus a couple of registry values) used to avoid hitting the database on every read in code paths that use **f-Get-Site-Value-TT**.

### 3.2 Load and Rebuild

| Routine | Purpose |
|---------|--------|
| **Load-TT-Site-Values** | Called once when **services.p** is run; fills **TT-Site-Value** from **SiteConfigOption** (and registry). |
| **f-Rebuild-Site-Value-TT-From-Site** | Re-scans **SiteConfigOption** and repopulates **TT-Site-Value** (no registry re-read). Used after **globalassigns.i** so the cache matches the DB after startup reads. |

**When rebuild runs:** At the end of **lib/globalassigns.i** (e.g. after login or when the app initializes), **f-Rebuild-Site-Value-TT-From-Site()** is called so the TT is in sync with **SiteConfigOption** for the rest of the session.

### 3.3 Reading Options

| Function | Source | Typical use |
|----------|--------|-------------|
| **f-get-session-parm**( *preproc-name*, *parm-name* ) | **SiteConfigOption** (direct FIND by OptionName) | Startup and any code that needs the current DB value. **preproc-name** can request type: e.g. `"LOG"` normalizes YES/TRUE and NO/FALSE to "YES"/"NO"; `"INT"`, `"CHR"`, etc. for trimming. Returns ? if option not found. |
| **f-Get-Site-Value-TT**( *OptionName*, *ConvertFunction*, *DefaultValue*, *OptionDesc*, *CreateSiteConfig* ) | **TT-Site-Value** first; if missing, uses *DefaultValue* and optionally creates **SiteConfigOption** (and **ConfigOptionMaster**) via **Create-SiteConfig** | Code that prefers the cache and is okay with auto-creation and default when the option is absent. |

So:

- **f-get-session-parm** = read from **SiteConfigOption** (authoritative DB).
- **f-Get-Site-Value-TT** = read from **TT-Site-Value**; if not in TT, create TT row from default and optionally create DB rows.

### 3.4 Writing Options

| Function | Effect |
|----------|--------|
| **f-set-session-parm**( *Parm-Name*, *Parm-Value* ) | Finds or creates **SiteConfigOption** by *Parm-Name*, sets **OptionValue** = *Parm-Value*, then calls **f-Set-Site-Value-TT** so **TT-Site-Value** is updated as well. |
| **f-Set-Site-Value-TT**( *OptionName*, *OptionValue*, *ConvertFunction*, *DefaultValue*, *OptionDesc*, *UpdateSiteConfig* ) | Updates or creates **TT-Site-Value** for *OptionName*. If *UpdateSiteConfig* is TRUE, also finds or creates **SiteConfigOption** and sets its **OptionValue**. |

So writes go to **both** the database and the TT when the appropriate flag is set, keeping them in sync.

### 3.5 Create-SiteConfig (Auto-Creation)

**Procedure:** **Create-SiteConfig** in `lib/services.p` (which calls **lib/create-sitecfg.p**).

- **Inputs:** OptionName, default value, option description.
- **Behavior:** If **SiteConfigOption** has no row for that **OptionName**, it **creates** one with **OptionValue** = default. If **ConfigOptionMaster** has no row for that option, it **creates** one (OptionName, OptionDescr, OptionValue, AcceptableValues = default, OperatorEdits = yes).
- Used when **f-Get-Site-Value-TT** is called with **CreateSiteConfig** = YES and the option is missing, and by **set-from-site-cfg.i** when the option is not found.

---

## 4. When Configuration Is Loaded

### 4.1 Startup / Login

1. **services.p** is run (e.g. from main or login flow). It defines **TT-Site-Value** and runs **Load-TT-Site-Values** (TT populated from **SiteConfigOption** + registry).
2. **lib/globalassigns.i** runs (e.g. after login). It assigns many **globals** (in **gLoginVars.i** and elsewhere) using **f-get-session-parm** for options such as:
   - **ScaleID**, **PlantID**, **ShiftID**, **Version**
   - **SkipZeroTares**, **MfgID**, **ModifyCustomer**, **ModifyKillDate**, **ModifyLot**, etc.
   - **FalseMidnight**, **FalseMidnightKill** (with validation and fallback to `"24:00"` if invalid)
   - **MaximumBackDate**, **MaximumForwardDate**, **Max/MinVarOffset**, **Max/MinKillOffset**
   - **WindowCol**, **WindowRow**, **LabelDirectory**, **GoalsSelectBatch**, and many more.
3. At the **end of globalassigns.i**, **f-Rebuild-Site-Value-TT-From-Site()** is run so **TT-Site-Value** matches **SiteConfigOption** for the rest of the session.

So “site configuration” for the session is effectively fixed after this point for options that were read into globals; other options are read on demand via **f-get-session-parm** or **f-Get-Site-Value-TT**.

### 4.2 After Editing in the Site Config Window

When the user edits options in the **Edit Site Config** window (**sobjects/s-sitecfg.w**) and quits, **btnQuit** runs **globalassigns.i** again and then **f-Rebuild-Site-Value-TT-From-Site()**, so:

- Globals are refreshed from the updated **SiteConfigOption**.
- **TT-Site-Value** is repopulated from the same.

So leaving the site config screen reapplies configuration for the session.

---

## 5. UI and Navigation

| Component | Purpose |
|-----------|--------|
| **sobjects/s-sitecfg.w** | “Edit Site Config” window; contains browser and viewer; **btnQuit** runs **globalassigns** + rebuild so changes take effect. |
| **browsers/b-sitecfg.w** | Browser over **SiteConfigOption** (and **ConfigOptionMaster**); can filter by OperatorEdits (operator-editable only). |
| **viewers/v-sitecfg.w** | Viewer for the selected option; shows **ConfigOptionMaster** fields and a combo for **AcceptableValues**; edits **SiteConfigOption.OptionValue**. |
| **dialogs/d-sitecfgfind.w** | Find/search dialog to locate an option by name (used from **btnFind** on the site config window). |

Options are **case-sensitive** by name. **AcceptableValues** in the viewer are pipe-separated; the UI often converts pipes to commas for combo list-items.

---

## 6. Relationship to Other Concepts

### 6.1 Operator / Login

- **PlantID**, **ShiftID**, **ScaleID**, and many behavior flags (e.g. **ModifyCustomer**, **Goal Select By Date**) come from site config and are applied at login via **globalassigns.i**.
- **Valid Departments** and **ValidShifts** are stored in **ConfigOptionMaster.AcceptableValues** (not **SiteConfigOption**); the login screen uses them as the list of allowed values for Department and Shift.

### 6.2 Serial, Labels, Weigh Flow

- **FalseMidnight** / **FalseMidnightKill** (from site config) drive pack/kill date adjustment in **createserial.p** and reporting.
- **SerialVerification**, **LabelDirectory**, **PrinterDefaultDevice**, and similar options affect serial creation, label path, and which device is used.
- **gDepartment** is set at login from the Department dropdown (fed by **ConfigOptionMaster "Valid Departments"**); it is not stored in **SiteConfigOption** as the “current” department—that is session state.

### 6.3 Modify, Goals, Product

- **Modify**-related permissions (e.g. **ModifyCustomer**, **ModifyLot**) and defaults come from site config.
- **ZuluCapture**, **ZuluFromGlobal**, **ZuluGlobalDeptShift1/2/3** are site options that control Zulu Dept behavior and storage (Modify.ModText1 NVP, CrossRef, etc.).
- **ProcessingMode**, **PgmSpecialProcessing**, **ModeProdCodeDefault** tie to product/process selection and default product.

### 6.4 Device / Scale / Printer

- **ScaleID**, **ScaleComPort**, **ScaleSpeed**, **ScaleParity**, **ScaleDataBits**, **ScaleStopBits**, **ScaleTimeOut**, **ScaleFlowControl** (and similar) define scale and communication settings.
- **PrinterComPort**, **PrinterSpeed**, **PrinterParity**, **PrinterDataBits**, **PrinterStopBits**, **PrinterDefaultDevice**, **PrinterFlowControl**, **PrinterDTREnable**, **PrinterOutBufferSize** define label printer behavior.

### 6.5 Host, MII, Messaging

- **HostType**, **PDNHostName**, **UploadSerials**, **UploadProdTotals**, **ResendSerialsRetryMins**, **ICICTSID**, and similar options control host integration and messaging.
- **FalseMidnight Set By Server** and MII-related options (e.g. **MII-PackDate**, **MII-ProcessOrder**) are stored as site options.

### 6.6 Goals, Batch, Cleansing

- **Clear-Goals-Days**, **Goals-Purge-Days**, **BatchOrder Clear Days**, **Clear-MfgOrd-Days** control how long data is kept.
- **Goal Select By Date**, **GoalsSelectOrder**, **GoalsSelectBatch**, and related globals drive how goals are chosen and displayed.

### 6.7 ConfigOptionMaster Without SiteConfigOption

- Some options are **definition-only**: e.g. **Valid Departments**, **ValidShifts**. The **list** of allowed values is in **ConfigOptionMaster.AcceptableValues**; the “current” value may be session state (e.g. **gDepartment**, **gShift**) rather than a row in **SiteConfigOption**.

---

## 7. Business Rules (Summary)

1. **OptionName** is the single key in both **ConfigOptionMaster** and **SiteConfigOption**; names are case-sensitive.
2. **AcceptableValues** (pipe-separated) define valid values for dropdowns and can be used for validation; **OperatorEdits** controls whether the option is editable in the operator site config UI.
3. **SiteConfigOption** holds one value per option per site; if a row is missing, the system may create it (and optionally a **ConfigOptionMaster** row) when a default and description are provided.
4. **f-get-session-parm** reads from **SiteConfigOption**; **f-Get-Site-Value-TT** reads from **TT-Site-Value** and can create DB and TT rows from a default.
5. **f-set-session-parm** and **f-Set-Site-Value-TT** (with **UpdateSiteConfig** = YES) write to both **SiteConfigOption** and **TT-Site-Value** so they stay in sync.
6. After **globalassigns.i** (startup or after quitting site config), **f-Rebuild-Site-Value-TT-From-Site** is run so the TT matches the database.

---

## 8. Option List and References

The full list of **118+ options** (option name, description, default, acceptable values, operator-editable) is in **[site-configuration-options.md](site-configuration-options.md)**. That document is the reference for “what options exist”; this document explains **how** site configuration works as a concept.

---

## 9. Redevelopment Notes

- **Two tables:** Keep **ConfigOptionMaster** (metadata + defaults) and **SiteConfigOption** (current value per option). Optionally support auto-creation of missing rows with a default and description.
- **Access pattern:** Provide a single “get option” API that can read from DB and/or an in-memory cache; on startup, load options that drive globals and refresh cache after bulk or UI-driven updates.
- **Writes:** When updating an option, update both persistent store and any in-memory cache so the session sees the new value without a full restart.
- **Rebuild:** After applying a batch of config (e.g. after “Edit Site Config” or login), refresh the cache from the database so all code using the cache sees the same state.
- **Valid lists:** Options like **Valid Departments** and **ValidShifts** that define **lists** can stay in **ConfigOptionMaster.AcceptableValues**; the “selected” value can remain session or login state if that matches current behavior.

---

**Related docs:** [CONCEPTS-AND-BUSINESS-RULES.md](CONCEPTS-AND-BUSINESS-RULES.md), [site-configuration-options.md](site-configuration-options.md) (option list), [FALSE-MIDNIGHT-CONCEPT-AND-USAGE.md](FALSE-MIDNIGHT-CONCEPT-AND-USAGE.md), [DEPARTMENT-CONCEPT-AND-USAGE.md](DEPARTMENT-CONCEPT-AND-USAGE.md), [MODIFY-CONCEPT-AND-USAGE.md](MODIFY-CONCEPT-AND-USAGE.md).
