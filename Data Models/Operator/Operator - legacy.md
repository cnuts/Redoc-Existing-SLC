
## 1. Overview

### What Is an Operator?

An **Operator** is a **user account** in the legacy SLC application. It represents a person who logs into the system to perform production activities like printing labels, weighing products, modifying production parameters, and managing goals.

| Concept             | Definition                                      | Example                                  |
| ------------------- | ----------------------------------------------- | ---------------------------------------- |
| **Operator**        | User login account with credentials             | `jsmith` (John Smith)                    |
| **Credentials**     | Username and password for authentication        | `jsmith` / `pass123`                     |
| **Operator Record** | Database entry with name, language, permissions | Stored in **Operator** table             |
| **Permissions**     | Functions/features operator is allowed to use   | `Print.LabelPrint`, `Print.Modify.Price` |
| **Session**         | Active login with operator context              | Active from login until logout/timeout   |

### Why Operators Exist

Manufacturing facilities need:

- **Accountability** - Know who produced/printed each unit
- **Security** - Control who can access which functions
- **Audit Trails** - Track user actions for compliance
- **Performance Metrics** - Measure output by operator
- **Multi-Language Support** - Each operator selects their language
- **Shift Assignment** - Track which shift operator is working

---

## 2. Table Structure

### Operator Table

**Primary Key:** `Operator` (login name)

```sql
CREATE TABLE Operator (
    -- Identification
    Operator           CHAR(10) NOT NULL PRIMARY KEY,  -- Login name (e.g., "jsmith")
    Password           CHAR(10) NOT NULL,              -- Encrypted password

    -- Personal Information
    FirstName          CHAR(20),                       -- First name (e.g., "John")
    LastName           CHAR(20),                       -- Last name (e.g., "Smith")

    -- Preferences
    LangCode           CHAR(10)                        -- Language preference (e.g., "EN", "ES")
);
```

**Index:** `Oper` (primary) on `Operator ASCENDING`

| Field         | Type        | Purpose                | Example                       | Notes                         |
| ------------- | ----------- | ---------------------- | ----------------------------- | ----------------------------- |
| **Operator**  | CHAR(10) PK | Login name             | `jsmith`, `mgarcia`, `test01` | Unique identifier             |
| **Password**  | CHAR(10)    | Encrypted password     | `*****`                       | Stored securely, max 10 chars |
| **FirstName** | CHAR(20)    | First name of operator | `John`, `Maria`               | For display/reports           |
| **LastName**  | CHAR(20)    | Last name of operator  | `Smith`, `Garcia`             | For display/reports           |
| **LangCode**  | CHAR(10)    | Language preference    | `EN`, `ES`, `FR`              | Determines UI language        |

### OperatorAppFunction Table

**Relationship:** Operator (1:N) ──→ OperatorAppFunction (N:M) ──→ AppFunction

Defines which functions each operator is permitted to perform.

```sql
CREATE TABLE OperatorAppFunction (
    Operator           CHAR(10) NOT NULL,              -- FK to Operator
    ID                 CHAR(50) NOT NULL,              -- FK to AppFunction
    Permitted          LOGICAL DEFAULT NO,             -- YES/NO permission

    PRIMARY KEY (Operator, ID)
);
```

**Indexes:**

- `OperatorID` (primary) on `Operator, ID`
- `ID-Operator` (unique non-primary) on `ID, Operator`

| Field         | Type        | Purpose                    | Example            |
| ------------- | ----------- | -------------------------- | ------------------ |
| **Operator**  | CHAR(10) FK | Which operator             | `jsmith`           |
| **ID**        | CHAR(50) FK | Which application function | `Print.LabelPrint` |
| **Permitted** | LOGICAL     | Is permission granted      | YES or NO          |

### AppFunction Table

**Purpose:** Master list of all functions/features in the application.

```sql
CREATE TABLE AppFunction (
    ID                 CHAR(50) NOT NULL PRIMARY KEY,  -- Function identifier
    Description        CHAR(60) NOT NULL               -- Human-readable description
);
```

**Index:** `ID-index` (primary) on `ID ASCENDING`

| Field           | Type        | Purpose                                | Example                                  |
| --------------- | ----------- | -------------------------------------- | ---------------------------------------- |
| **ID**          | CHAR(50) PK | Function identifier (period-separated) | `Print.LabelPrint`, `Print.Modify.Price` |
| **Description** | CHAR(60)    | Description for UI/reports             | `Allow printing labels`                  |

#### Common AppFunction Examples

```
Print.LabelPrint                    → Print case/item labels
Print.Modify.Price                  → Edit price on modify screen
Print.Modify.Lot                    → Edit lot number on modify screen
Print.Modify.PackDate               → Edit pack date offset
Print.Modify.KillDate               → Edit kill date offset
Print.Modify.VarOffset              → Edit variable offset
Print.Modify.Tare                   → Edit tare number
Print.Modify.ZuluCapture            → Edit Zulu/NVP data
Setup.Goals                         → Create/modify production goals
Setup.Products                      → Manage product definitions
Setup.Operators                     → Create/modify operators and permissions
System.Admin                        → System administration functions
System.Config                       → Edit system configuration
Reports.Production                  → View production reports
Reports.Quality                     → View quality/audit trail reports
```

---

## 3. Operator Lifecycle

### Login Flow

```
1. User enters credentials (username/password)
   ↓
2. System authenticates against Operator table
   ├─ Password matches → success
   └─ Password fails → error, retry
   ↓
3. System loads Operator record
   ├─ FirstName, LastName
   ├─ LangCode (set UI language)
   └─ Operator ID (set login context)
   ↓
4. System loads OperatorAppFunction permissions
   ├─ Query all AppFunction IDs where Permitted=YES
   ├─ Build permission set for this operator
   └─ Store in session/global (g-OperatorPermissions)
   ↓
5. User presented with available functions/screens
   └─ Only functions where Permitted=YES are enabled
   ↓
6. User performs production work
   ├─ All actions tracked with Operator ID
   ├─ Serial records capture Operator field
   └─ Audit trails log user actions
   ↓
7. User logs out or session times out
   └─ Session cleared, operator context released
```

### Session Management

During an active session:

```progress
/* Global operator context */
gOperator          /* CHAR(10) - logged-in operator ID */
gOperatorName      /* CHAR(50) - "FirstName LastName" for display */
gLangCode          /* CHAR(10) - UI language */
gOperatorPermissions /* Character string or table of permitted function IDs */

/* Used for audit trail and accountability */
/* Every Serial record created has Operator = gOperator at creation time */
/* Every logged action includes timestamp + gOperator for audit trail */
```

---

## 4. Operator and Permissions System

### Permission Gate Logic

When a user attempts to perform a function:

```progress
/* Pseudo-code: Check if operator has permission */

DEFINE VARIABLE v-FunctionID AS CHARACTER NO-UNDO VALUE "Print.LabelPrint".

FOR FIRST OperatorAppFunction NO-LOCK
    WHERE OperatorAppFunction.Operator = gOperator
      AND OperatorAppFunction.ID = v-FunctionID
      AND OperatorAppFunction.Permitted = YES:

    /* Permission granted - allow function */
    ENABLE-ALL-BUTTONS.

END.

IF NOT AVAILABLE OperatorAppFunction THEN DO:
    /* Permission denied - disable function or show error */
    DISABLE-ALL-BUTTONS.
    MESSAGE "You do not have permission to print labels" VIEW-AS ALERT-BOX.
END.
```

### Permission Examples

**Operator: jsmith**

| Function                | Permitted | Effect                            |
| ----------------------- | --------- | --------------------------------- |
| `Print.LabelPrint`      | YES       | Can print labels ✓                |
| `Print.Modify.Price`    | YES       | Can edit price on modify screen ✓ |
| `Print.Modify.Lot`      | YES       | Can edit lot number ✓             |
| `Print.Modify.KillDate` | NO        | Cannot edit kill date ✗           |
| `Setup.Goals`           | NO        | Cannot create goals ✗             |

**Operator: manager01**

| Function           | Permitted | Effect                       |
| ------------------ | --------- | ---------------------------- |
| `Print.LabelPrint` | YES       | Can print labels ✓           |
| `Print.Modify.*`   | YES       | Can edit all modify fields ✓ |
| `Setup.Goals`      | YES       | Can create/modify goals ✓    |
| `Setup.Operators`  | YES       | Can manage operators ✓       |
| `System.Config`    | YES       | Can edit system config ✓     |

---

## 5. Operator and Serial Records

### Accountability Through Operator Field

Every **Serial** record has an **Operator** field that captures who printed the label:

```sql
Serial {
    SerialNum           CHAR(12),
    Operator            CHAR(10),       -- Who printed this label
    PrintDate           DATE,           -- When it was printed
    ProductCode         INT,
    ...
}
```

#### Serial Creation Flow

```
1. Operator logs in
   → gOperator = "jsmith"

2. Operator scans product barcode
   → System loads product configuration

3. Operator places case on scale
   → System captures weight

4. Operator presses "Print Label"
   → System calls CreateSerial()

5. CreateSerial() executes
   ├─ Creates Serial record
   ├─ Assigns: Serial.Operator = gOperator  ← AUDIT TRAIL
   ├─ Assigns: Serial.PrintDate = TODAY
   ├─ Assigns: Serial.PrintTime = NOW (seconds)
   ├─ Stores: Product, weights, pack date, kill date
   └─ Returns: SerialNum for label printing

6. System prints label with SerialNum
   → Label contains:
      ├─ SerialNum (barcode)
      ├─ Product name
      ├─ Weight
      ├─ Pack date
      ├─ Kill date
      └─ (Operator not typically on label but in database)

7. Operator places label on case, ships
   → Case tracked through Serial record

8. Later: Quality issue investigation
   → Query: WHERE Serial.Operator = 'jsmith' AND Serial.PrintDate = '03/11/2026'
   → Find all cases printed by jsmith on that date
   → Analyze for quality patterns
```

#### Query Example: Operator Accountability

**Scenario:** Product recall - find all units produced by specific operator on specific date

```sql
SELECT SerialNum, ProductCode, PrintTime, NetWgt, Quality_Status
FROM Serial
WHERE Operator = 'jsmith'
  AND PrintDate = '2026-03-11'
  AND Shift IN (1, 2)
ORDER BY PrintTime;
```

**Result:** Shows exactly which units (serials) that operator produced, enabling targeted recall/investigation.

---

## 6. Operator and Production Goals

### Operator Context During Goal Production

When producing to a **Goal** (target production):

```
Goal: "Produce 1000 cases of Product 1001 by 3:00 PM"

1. Goal is created and assigned to scale/shift
2. Operators on that shift log in
3. System shows: "Active Goal - Target 1000 cases"
4. Operators print labels and produce cases
5. Each Serial created:
   ├─ Operator = logged-in operator ID (gOperator)
   ├─ GoalID = active goal ID
   ├─ ProductCode = goal's product
   └─ Contribution = 1 case toward goal
6. System updates goal progress:
   └─ CasesProduced = COUNT(Serial WHERE GoalID = 5001)
7. Operator sees: "500 of 1000 cases completed (50%)"
8. When goal reaches 1000 cases: Goal status changes to ACHIEVED
```

### Performance Metrics by Operator

System can generate reports:

```
Daily Production Summary by Operator (Date: 03/11/2026)

Operator: jsmith
  - Total Cases Printed: 245
  - Total Weight: 612.5 lbs
  - Avg Weight: 2.50 lbs/case
  - Shift(s): 1, 2
  - Goals Contributed To: 3 (Goal 5001: 113 cases, Goal 5002: 80 cases, Goal 5003: 52 cases)

Operator: mgarcia
  - Total Cases Printed: 189
  - Total Weight: 472.5 lbs
  - Avg Weight: 2.50 lbs/case
  - Shift(s): 2, 3
  - Goals Contributed To: 2 (Goal 5002: 95 cases, Goal 5003: 94 cases)
```

---

## 7. Operator and Shifts

### Shift Context

**Shift** is the time period when operator is working:

```
Shift Configuration:
  Shift 1: 6:00 AM - 2:00 PM
  Shift 2: 2:00 PM - 10:00 PM
  Shift 3: 10:00 PM - 6:00 AM
```

### Operator Shift Assignment

Operators are typically assigned to specific shifts:

```
jsmith:
  - Assigned to Shift 1 (6 AM - 2 PM)
  - Regular production operator
  - 40 hours/week

mgarcia:
  - Assigned to Shift 2 (2 PM - 10 PM) and Shift 3 (10 PM - 6 AM)
  - Rotating shifts
  - On-call for coverage

manager01:
  - Covers all shifts as needed
  - Supervisory role
  - Flexible schedule
```

### Login - Shift Selection

When operator logs in at login screen:

```
Login Screen:
  Operator: [jsmith____]
  Password: [****]
  Shift:    [1___]  ← User selects their shift for this session

After Login:
  gOperator = "jsmith"
  gShift = 1
  gShiftStart = "06:00"
  gShiftEnd = "14:00"

During Session:
  All Serial records created: Serial.Shift = gShift (1)
  All reports filtered: WHERE Shift = 1
  Configuration adjusted: Zulu dept by shift (gShift)
```

### Shift Tracking in Serial

Each Serial records which shift it was produced under:

```
Serial {
    SerialNum = "20260311-001",
    Operator = "jsmith",
    Shift = 1,      -- Morning shift (6 AM - 2 PM)
    PrintDate = 03/11/2026,
    ...
}

Serial {
    SerialNum = "20260311-102",
    Operator = "mgarcia",
    Shift = 2,      -- Afternoon shift (2 PM - 10 PM)
    PrintDate = 03/11/2026,
    ...
}
```

### Shift Reports

**Query:** All units produced by operator on their shift:

```sql
SELECT SerialNum, PrintTime, ProductCode, NetWgt
FROM Serial
WHERE Operator = 'jsmith'
  AND Shift = 1
  AND PrintDate = '2026-03-11'
ORDER BY PrintTime;
```

---

## 8. Relationships to Other Concepts

### Operator ← Serial (One-to-Many)

```
Operator (1:N) ──→ Serial

One Operator can print/create MANY Serial records
Each Serial has AT MOST ONE Operator

Example:
  Operator "jsmith"
    → Created Serial ABC-001, ABC-002, ABC-003, ... (hundreds of serials)
    → Each serial marked with Operator = "jsmith"
```

**Use Case:** Quality investigation

```
Problem: Metal detected in product from batch ABC
Solution Query:
  SELECT DISTINCT Operator, COUNT(*) as CasesCount
  FROM Serial
  WHERE ProductCode = 1001
    AND PrintDate = '2026-03-10'
  GROUP BY Operator

Result: jsmith (245 cases), mgarcia (189 cases)
Next: Check jsmith's cases for defect patterns
```

### Operator ← OperatorAppFunction ← AppFunction (Many-to-Many)

```
Operator (1:N) ──→ OperatorAppFunction (N:M) ──→ AppFunction
```

Links operators to their permitted functions/features.

**Example Permission Structure:**

```
Operator "jsmith":
  ├─ Print.LabelPrint (Permitted = YES)
  ├─ Print.Modify.Price (Permitted = YES)
  ├─ Print.Modify.Lot (Permitted = YES)
  ├─ Print.Modify.KillDate (Permitted = YES)
  ├─ Setup.Goals (Permitted = NO)
  └─ System.Config (Permitted = NO)

Operator "manager01":
  ├─ Print.* (Permitted = YES for all Print functions)
  ├─ Setup.* (Permitted = YES for all Setup functions)
  ├─ System.* (Permitted = YES for all System functions)
  └─ Reports.* (Permitted = YES for all Reports functions)
```

**Permission Gate Example:**

When operator clicks "Modify Price" button:

```progress
FIND OperatorAppFunction WHERE
    Operator = gOperator AND
    ID = "Print.Modify.Price" AND
    Permitted = YES
    NO-ERROR.

IF AVAILABLE OperatorAppFunction THEN
    /* Show modify screen with price field enabled */
    DISPLAY ModifyScreen WITH BUTTON "Price" ENABLED.
ELSE
    /* Hide or disable price field */
    MESSAGE "You do not have permission to modify price" VIEW-AS ALERT-BOX.
END.
```

### Operator ← Goal (No Direct Link, Contextual)

```
Operator (implicit context) ──→ Goal (association via Serial)

Goals are achieved by operators producing units:
  Goal 5001 (produce 1000 cases)
    → Serial 001 (Operator="jsmith", GoalID=5001)
    → Serial 002 (Operator="jsmith", GoalID=5001)
    → Serial 003 (Operator="mgarcia", GoalID=5001)
    ...
```

**Use Case:** Goal progress by operator

```sql
SELECT
    g.GoalID,
    g.TargetLevel,
    o.Operator,
    COUNT(s.SerialNum) as ProducedByOperator,
    ROUND(100.0 * COUNT(s.SerialNum) / g.TargetLevel, 2) as PercentContribution
FROM Goals g
LEFT JOIN Serial s ON g.GoalID = s.GoalID
LEFT JOIN Operator o ON s.Operator = o.Operator
WHERE g.GoalID = 5001
GROUP BY g.GoalID, g.TargetLevel, o.Operator
ORDER BY ProducedByOperator DESC;

Result:
GoalID | Target | Operator | Produced | Percent
5001   | 1000   | jsmith   | 450      | 45.0%
5001   | 1000   | mgarcia  | 380      | 38.0%
5001   | 1000   | test01   | 170      | 17.0%
```

### Operator ← Modify (Indirect, Permission-Based)

```
Operator [Permissions via OperatorAppFunction]
    ↓ (Print.Modify.* permissions)
    → Can Edit Modify Screen Fields

Example:
  Operator "jsmith" has "Print.Modify.Price" = YES
    → Price field is ENABLED on modify screen
    → jsmith can change price for product before printing

  Operator "jsmith" has "Print.Modify.KillDate" = NO
    → Kill Date field is DISABLED on modify screen
    → jsmith cannot change kill date (only manager can)
```

### Operator ← Shift

```
Operator --assigned to--> Shift

When operator logs in:
  gOperator = "jsmith"
  gShift = 1        ← Selected during login or from default assignment

All production ties to operator + shift:
  Serial.Operator = "jsmith"
  Serial.Shift = 1

Enables: Shift-specific reports, performance analysis by operator & shift
```

### Operator ← Language (Multi-Language Support)

```
Operator.LangCode → System displays UI in that language

Example:
  Operator "jsmith" → LangCode = "EN" → English UI
  Operator "mgarcia" → LangCode = "ES" → Spanish UI

When operator logs in:
  gLangCode = Operator.LangCode
  → All screens, messages, buttons display in that language
```

### Operator ← Audit Trails & Accountability

```
Core Purpose: Track WHO did WHAT and WHEN

Example Audit Entry:
  Timestamp: 2026-03-11 14:35:22
  Operator: jsmith
  Action: Print Label
  Serial: ABC-123-001
  Product: 1001 (Milk 2L)
  Status: SUCCESS

Used For:
  - Quality investigations
  - Performance metrics
  - Compliance/regulatory audits
  - User activity logs
```

---

## 9. Configuration Examples

### Example 1: Standard Production Facility

**Facilities:** 3 shifts, multiple operators per shift

```
Operator: jsmith (John Smith)
  ├─ Password: securepass123
  ├─ FirstName: John
  ├─ LastName: Smith
  ├─ LangCode: EN
  └─ Permissions:
      ├─ Print.LabelPrint = YES
      ├─ Print.Modify.Price = YES
      ├─ Print.Modify.Lot = YES
      ├─ Print.Modify.PackDate = YES
      ├─ Print.Modify.KillDate = NO
      ├─ Setup.Goals = NO
      └─ System.Config = NO

Operator: mgarcia (Maria Garcia)
  ├─ Password: maria9876
  ├─ FirstName: Maria
  ├─ LastName: Garcia
  ├─ LangCode: ES
  └─ Permissions:
      ├─ Print.LabelPrint = YES
      ├─ Print.Modify.Price = YES
      ├─ Print.Modify.Lot = YES
      ├─ Print.Modify.PackDate = YES
      ├─ Print.Modify.KillDate = NO
      ├─ Setup.Goals = NO
      └─ System.Config = NO

Operator: manager01 (Production Manager)
  ├─ Password: manager1234
  ├─ FirstName: Bob
  ├─ LastName: Johnson
  ├─ LangCode: EN
  └─ Permissions:
      ├─ Print.* = YES (all print functions)
      ├─ Setup.Goals = YES
      ├─ Setup.Products = YES
      ├─ Setup.Operators = YES
      └─ System.Config = YES
```

**Production Workflow:**

```
6:00 AM: jsmith logs in (Shift 1)
  → All labels printed between 6 AM - 2 PM marked with Operator="jsmith", Shift=1

2:00 PM: mgarcia logs in (Shift 2)
  → All labels printed between 2 PM - 10 PM marked with Operator="mgarcia", Shift=2

10:00 PM: jsmith logs in again (Shift 3)
  → All labels printed between 10 PM - 6 AM marked with Operator="jsmith", Shift=3

manager01: Logs in as needed for supervision, goal creation, operator edits
```

---

### Example 2: Small Facility with Limited Operators

**Setup:** 1-2 operators, simple permission model

```
Operator: test01
  ├─ Password: test123
  ├─ FirstName: Test
  ├─ LastName: User
  ├─ LangCode: EN
  └─ Permissions: (All enabled for testing/training)
      ├─ Print.* = YES
      ├─ Setup.* = YES
      ├─ System.* = YES
      └─ Reports.* = YES
```

**Use Case:** Training environment, testing, small startup

---

### Example 3: Multi-Site with Different Permission Levels

**Setup:** 2 facilities with different operator access

**Site A (Miami):**

```
Operators:
  - jsmith (Full print permissions)
  - mgarcia (Full print permissions)
  - miami_manager (Admin permissions)
```

**Site B (NewYork):**

```
Operators:
  - akim (Full print permissions)
  - rrodriguez (Full print permissions)
  - ny_manager (Admin permissions)
```

**Each operator:** Only sees their site's data, can't access other site's production

---

## 10. Operator Management Operations

### Creating a New Operator

**Screen:** sobjects/s-operatorsecurity.w

```
Input:
  Operator ID: jsmith
  Password: initialpass123
  First Name: John
  Last Name: Smith
  Language: EN

Process:
1. Validate Operator ID not already in use
2. Encrypt password
3. Create Operator record
4. Show permission assignment screen

After Creation:
  Create placeholder OperatorAppFunction records
  Each with Permitted = NO initially
  Manager then selects YES for needed permissions
```

### Modifying Operator Permissions

**Screen:** browsers/b-operatorappfunction.w

```
Select Operator: [jsmith___________]

Available Functions:
  ☑ Print.LabelPrint
  ☑ Print.Modify.Price
  ☑ Print.Modify.Lot
  ☐ Print.Modify.KillDate
  ☐ Setup.Goals
  ☐ System.Config

Save → Updates OperatorAppFunction table
```

### Changing Operator Password

```
Screen prompts:
  Current Password: [***]
  New Password:     [***]
  Confirm:          [***]

Validation:
  - Current password matches
  - New password ≠ current
  - New password meets complexity rules

Update:
  Operator.Password = encrypt(new_password)
```

### Disabling an Operator

```
Mark Operator as inactive (logical delete):
  UPDATE Operator
  SET Password = ''  /* or add IsActive = NO field */
  WHERE Operator = 'jsmith'

Effect:
  - Operator cannot log in
  - Historical Serial records still show operator
  - Audit trail preserved
```

---

## 11. Security Considerations

### Password Storage

```
Requirement: Passwords must be encrypted
Implementation: Use Progress encryption functions
Example:
  ASSIGN Operator.Password = ENCODE(input-password)  /* Encrypt on store */
  ASSIGN plaintext = DECODE(Operator.Password)       /* Decrypt on verify */
```

### Session Timeout

```
Session Starts:
  Login time recorded
  gLoginTime = NOW

During Session:
  Check: IF (NOW - gLoginTime) > SESSION-TIMEOUT THEN
    → Auto-logout, clear gOperator context
    → User must log in again

Configuration:
  SessionTimeoutMinutes = 15  (typically 15-30 minutes)
```

### Audit Trail

```
Every operator action logged:
  - Login time/date
  - Functions accessed
  - Serial records created (operator tracked automatically)
  - Data modifications
  - Logout time/date

Enables:
  - Compliance with food safety regulations
  - Investigation of issues/anomalies
  - Performance tracking
```

### Permission Hierarchies

**Role-Based Model:**

```
Role: Production Operator
  Permissions:
    - Print.LabelPrint
    - Print.Modify.Price
    - Print.Modify.Lot
    - Reports.ProductionDay

Role: Shift Supervisor
  Permissions:
    - (All Production Operator permissions)
    - Setup.Goals
    - Print.Modify.KillDate
    - Reports.QualityIssues

Role: System Administrator
  Permissions:
    - (All Permissions)
    - System.Config
    - Setup.Operators
    - Backup/Restore
```

---

## 12. Common Questions & Troubleshooting

### Q: An operator forgot their password - what do we do?

**A:**

```
Option 1: Reset to temporary password
  1. Admin goes to operator management screen
  2. Finds operator record
  3. Sets temporary password (e.g., "temp1234")
  4. Operator logs in with temporary password
  5. System forces password change on first login
  6. Operator creates new secure password

Option 2: Clear password and require reset
  UPDATE Operator
  SET Password = ''
  WHERE Operator = 'jsmith'
  /* Next login required system-generated temporary password */
```

### Q: Can one operator have multiple logins?

**A:** No, not simultaneously. An operator can:

```
✓ Log in from different shifts as different shift contexts
  └─ jsmith logs in Shift 1 (6 AM - 2 PM)
  └─ jsmith logs in Shift 3 (10 PM - 6 AM, same day)
  Side effect: Serial records track both as Operator="jsmith" but different Shift

✓ Log in from different physical locations/scales
  └─ Single Operator account can be used at multiple scales
  └─ Each login is a separate session

✗ Two simultaneous logins of same account
  └─ Second login forces first logout
  └─ Only one session per operator at a time
```

### Q: Why would we need multiple language operators?

**A:**

```
Bilingual/Multilingual Facility:
  - Hiring local workforce (Spanish, Portuguese, etc.)
  - International manufacturing site
  - Immigrant communities

Example:
  Operator "jsmith" (English-preferred) → LangCode = "EN"
  Operator "mgarcia" (Spanish-preferred) → LangCode = "ES"

When each logs in:
  - UI displays in their preferred language
  - Error messages, buttons, menus translated
  - No confusion with mixed-language production
```

### Q: How do we track which operator created a serial?

**A:** Serial record has Operator field:

```sql
/* Query: All serials from specific operator today */
SELECT SerialNum, ProductCode, PrintTime, NetWgt
FROM Serial
WHERE Operator = 'jsmith'
  AND PrintDate = TODAY;

/* Query: Which operators produced goal 5001 */
SELECT DISTINCT s.Operator, COUNT(*) as SerialCount
FROM Serial s
WHERE s.GoalID = 5001
GROUP BY s.Operator
ORDER BY SerialCount DESC;
```

### Q: Can we prevent operator from editing certain fields?

**A:** Yes, through permissions:

```
Operator "jsmith":
  Print.Modify.Price = YES       → Can edit price ✓
  Print.Modify.KillDate = NO     → Cannot edit kill date ✗

On modify screen:
  Price field:     ENABLED (jsmith can enter)
  Kill Date field: DISABLED (jsmith cannot touch)
```

---


---

## 14. Operator Configuration Checklist

### Setting Up a New Operator

- [ ] Determine operator ID (username, max 10 chars)
- [ ] Create strong initial password
- [ ] Enter first and last name for identification
- [ ] Select preferred language (EN, ES, FR, etc.)
- [ ] Identify shift assignments (1, 2, 3)
- [ ] Identify required permissions:
  - [ ] Print functions (Print.LabelPrint, etc.)
  - [ ] Modify permissions (Print.Modify.\*, etc.)
  - [ ] Setup permissions (if supervisory role)
  - [ ] System permissions (if administrative role)
  - [ ] Report access (Reports.\*, etc.)
- [ ] Create Operator record in database
- [ ] Assign OperatorAppFunction records with correct permissions
- [ ] Test login with temporary password
- [ ] Force password change on first login
- [ ] Verify correct screens/functions are enabled
- [ ] Document operator role and responsibilities

### Ongoing Operator Maintenance

- [ ] Regular password resets (every 90 days recommended)
- [ ] Audit permission levels quarterly (remove unneeded, add if promoted)
- [ ] Disable operators who leave company
- [ ] Monitor audit trails for unusual activity
- [ ] Review operator performance metrics monthly
- [ ] Keep operator directory current (FirstName, LastName, LangCode)

---

## 15. Summary

| Aspect                | Details                                                                                               |
| --------------------- | ----------------------------------------------------------------------------------------------------- |
| **Purpose**           | User authentication, authorization, and accountability in production                                  |
| **Primary Table**     | Operator (username, password, name, language)                                                         |
| **Permissions**       | Through OperatorAppFunction (many-to-many with AppFunction)                                           |
| **Audit Trail**       | Serial records capture Operator field for traceability                                                |
| **Shift Context**     | Operator assigned to shifts for production timing                                                     |
| **Session**           | Active login with operator context, global variables (gOperator, gShift, etc.)                        |
| **Security**          | Encrypted passwords, session timeout, permission gates, audit logs                                    |
| **Key Relationships** | Serial (accountability), OperatorAppFunction (permissions), Shift (timing), Goal (production context) |
| **Configuration**     | System Admin creates operators, assigns permissions via UI                                            |
| **Compliance**        | Enables regulatory traceability for food safety (FDA, USDA)                                           |
| **Scope**             | Applies to all SLC v6.2+ system authentication and access control                                     |

---

