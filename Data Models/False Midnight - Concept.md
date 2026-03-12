
## 1. Overview

### What Is False Midnight?

**False Midnight** (FM) is a configurable time boundary that redefines when the production "day" begins and ends, distinct from calendar midnight (00:00). Instead of the calendar day running from 12:00 AM to 11:59 PM, an FM-configured system defines a "work day" that may run from 2:00 AM to 1:59 AM the next calendar day (or 10:00 PM to 9:59 PM, etc.).

| Concept            | Definition                           | Example                                |
| ------------------ | ------------------------------------ | -------------------------------------- |
| **Calendar Day**   | 00:00 to 23:59 (standard)            | Jan 15, 00:00 - Jan 15, 23:59          |
| **Work Day (FM)**  | FM time to FM time next calendar day | Jan 14, 2:00 AM - Jan 15, 1:59 AM      |
| **False Midnight** | Configured boundary time             | 2:00 AM, 10:00 PM, or 24:00 (disabled) |

### Why False Midnight Exists

Manufacturing facilities often operate on shifts that don't align with calendar midnight. For example:

- **Night Shift:** 10:00 PM - 6:00 AM (spans calendar days)
- **Day Shift:** 6:00 AM - 2:00 PM (spans calendar days during prep/cleanup)
- **Evening Shift:** 2:00 PM - 10:00 PM (spans calendar days during setup)

Without False Midnight:

- ❌ Night shift work split across two "days" (9/15 and 9/16)
- ❌ Production reports show split numbers (180 cases on 9/15, 220 on 9/16)
- ❌ Shift totals don't match actual production

With False Midnight (e.g., 2:00 AM):

- ✅ Night shift (10:00 PM - 6:00 AM) appears as single "work day"
- ✅ Production reports consolidate to one daily figure
- ✅ Shift totals match management expectations

### Business Impact

**Without FM:**

```
Calendar View:
  Sept 15: 180 cases (10 PM - midnight)
  Sept 16: 220 cases (midnight - 6 AM)
  Total: 400 cases (split across days - confusing)
```

**With FM = 2 AM:**

```
Work Day View:
  Sept 15 (2 AM - 1:59 AM next day): 400 cases
  Entire night shift = one "day" (clear production picture)
```

---

## 2. Configuration

### 2.1 Storage: SiteConfigOption Table

False Midnight is stored in the **SiteConfigOption** table as system configuration:

| Field           | Value                                                |
| --------------- | ---------------------------------------------------- |
| **OptionName**  | `FalseMidnight` (for PackDate/PrintDate adjustments) |
| **OptionName**  | `FalseMidnightKill` (for KillDate adjustments)       |
| **OptionValue** | Time in HH:MM format or "24:00" (disabled)           |
| **Description** | "False Midnight" or similar                          |

### 2.2 Valid Configuration Values

| Value     | Meaning                               | Use Case                                            |
| --------- | ------------------------------------- | --------------------------------------------------- |
| `24:00`   | Disabled (standard calendar midnight) | Standard calendar-aligned facilities                |
| `02:00`   | 2:00 AM shift boundary                | Morning shift facilities (night shift 10 PM - 2 AM) |
| `22:00`   | 10:00 PM shift boundary               | Evening shift facilities                            |
| `06:00`   | 6:00 AM shift boundary                | Day shift facilities                                |
| `14:00`   | 2:00 PM shift boundary                | Afternoon shift facilities                          |
| Any HH:MM | Custom time boundary                  | Site-specific shift patterns                        |

### 2.3 Internal Representation

Internally, FM time is converted to **seconds past midnight**:

```progress
v-FMTime = INT(SUBSTRING(OptionValue, 1, 2)) * 3600    /* hours * 3600 sec/hr */
         + INT(SUBSTRING(OptionValue, 4, 2)) * 60      /* minutes * 60 sec/min */
```

**Examples:**

- `02:00` → (2 × 3600) + (0 × 60) = **7,200 seconds**
- `10:00` → (10 × 3600) + (0 × 60) = **36,000 seconds**
- `22:00` → (22 × 3600) + (0 × 60) = **79,200 seconds**

### 2.4 Two FM Configurations: PackDate vs. KillDate

The system supports **two separate FM settings**:

**`FalseMidnight`** - Adjusts PackDate and PrintDate

- Used for production date (when case was packed)
- Used for label printing date
- Controls "work day" for production reporting

**`FalseMidnightKill`** - Adjusts KillDate

- Used for kill/process date
- Used for expiration date calculations
- Allows different FM boundary for lifecycle dates

**Why Two?** Different business needs:

- Production shift (2 AM) defines "work day"
- Kill/expiration shift (10 PM) defines "shelf life day"

---

## 3. Impact on Date Calculations

### 3.1 Single FM Calculation Logic

When FM is configured (not 24:00), date adjustments depend on **whether FM is before or after noon**.

#### Case 1: FM Before Noon (e.g., 2:00 AM)

**Scenario:** Report date = Sept 15, FM = 2:00 AM

The work day for "Sept 15" runs from **2:00 AM Sept 15 to 1:59 AM Sept 16**

**Calculation:**

- If current time ≥ 2:00 AM: Use current date (Sept 15)
- If current time < 2:00 AM: Use previous date (Sept 14)

**Example Timeline:**

```
Sept 14, 10:00 PM - Report date considered: Sept 14 (before 2 AM)
Sept 15, 12:00 AM - Report date considered: Sept 14 (before 2 AM)
Sept 15, 02:00 AM - Report date considered: Sept 15 (at/after 2 AM) ← FM BOUNDARY
Sept 15, 06:00 PM - Report date considered: Sept 15 (after 2 AM)
Sept 16, 01:00 AM - Report date considered: Sept 15 (before 2 AM)
Sept 16, 02:00 AM - Report date considered: Sept 16 (at/after 2 AM)
```

#### Case 2: FM After Noon (e.g., 10:00 PM)

**Scenario:** Report date = Sept 15, FM = 10:00 PM

The work day for "Sept 15" runs from **10:00 PM Sept 14 to 9:59 PM Sept 15**

**Calculation:**

- If current time ≥ 10:00 PM: Already in work day (current date)
- If current time < 10:00 PM: Use current date up to 10 PM (then resets tomorrow)

**Example Timeline:**

```
Sept 14, 06:00 PM - Report date considered: Sept 14 (before 10 PM)
Sept 14, 10:00 PM - Report date considered: Sept 14 (at 10 PM) ← FM BOUNDARY
Sept 14, 11:59 PM - Report date considered: Sept 14 (after 10 PM)
Sept 15, 06:00 AM - Report date considered: Sept 15 (same calendar day)
Sept 15, 09:59 PM - Report date considered: Sept 15 (before FM boundary)
Sept 15, 10:00 PM - Report date considered: Sept 15 (at FM; still current work day; boundary is exclusive for “next day”)
Sept 15, 10:00:01 PM - Report date considered: Sept 16 (new work day starts; code uses time &gt; FM, not ≥)
```

### 3.2 Application to Serial Records

When a **Serial record is created**, its dates are adjusted based on FM:

| Field         | Origin                 | FM Adjustment           | Purpose                               |
| ------------- | ---------------------- | ----------------------- | ------------------------------------- |
| **PackDate**  | Actual production time | Yes (FalseMidnight)     | "Work day" when case was packed       |
| **PrintDate** | Label print time       | Yes (FalseMidnight)     | "Work day" when label was printed     |
| **KillDate**  | Kill/process time      | Yes (FalseMidnightKill) | "Work day" for expiration calculation |
| **DateCode**  | Derived from PackDate  | Inherits PackDate FM    | Printed on label                      |

### 3.3 Effect on Reporting Filters

**Pack Date Report with FM = 2 AM:**

When user selects "Sept 15" for date range:

```progress
FOR EACH Serial NO-LOCK
    WHERE Serial.PackDate >= 2024-09-15 AND Serial.PackDate <= 2024-09-15
```

This query returns:

- Cases produced Sept 15, 2:00 AM - Sept 16, 1:59 AM (all marked PackDate = Sept 15)
- Includes night shift work (10 PM - 2 AM) from previous night
- Excludes early morning work after 2 AM that belongs to next FM "day"

---

## 4. Relationship to Other Concepts

### 4.1 Shifts & Work Days

**Relationship: FM defines the boundary between shifts**

| Element              | Without FM                           | With FM (2 AM)            | Impact                        |
| -------------------- | ------------------------------------ | ------------------------- | ----------------------------- |
| **Night Shift**      | 10 PM - 6 AM spans two calendar days | Single "work day"         | Clearer shift reporting       |
| **Shift Report**     | Numbers split across dates           | Consolidated by FM day    | Better management visibility  |
| **Shift Boundaries** | Calendar midnight (00:00)            | FM time (2:00 AM example) | Aligns with actual operations |

**Example:**

```
Night Shift (10 PM - 6 AM):
  Without FM: 100 cases (10 PM-midnight) + 150 cases (midnight-6 AM) = Split
  With FM=2 AM: 250 cases (single work day) = Clear total
```

### 4.2 Production Dates (PackDate, PrintDate, KillDate)

**Relationship: FM adjusts HOW dates are assigned**

When a case is produced at **11:45 PM with FM = 2 AM**:

```
Actual Calendar Time: Sept 15, 11:45 PM
FM Calculation: 11:45 PM > 2:00 AM? Yes, but on Sept 15
Previous day's FM?  2 AM on Sept 15 already passed
Current date's FM?  Next FM is Sept 16, 2:00 AM (hasn't arrived yet)
→ PackDate assigned: Sept 15 (current FM "work day")
```

When a case is produced at **1:30 AM with FM = 2 AM**:

```
Actual Calendar Time: Sept 16, 1:30 AM
FM Calculation: 1:30 AM < 2:00 AM? Yes, haven't hit FM yet
Previous FM boundary: Sept 15, 2:00 AM (already passed)
→ PackDate assigned: Sept 15 (still within yesterday's FM work day)
```

**Effect on Date Fields:**

- **PackDate:** FM-adjusted (work day date)
- **PrintDate:** FM-adjusted (when label printed, still in work day)
- **KillDate:** FM-adjusted (separate FalseMidnightKill setting)

### 4.3 Reporting Types & FM Application

**PackDate Reports (default):**

- Filters by FM-adjusted PackDate
- Shows consolidated "work day" totals
- Example: "Sept 15" actually includes 2 AM Sept 15 - 1:59 AM Sept 16

**PrintDate Reports:**

- Filters by FM-adjusted PrintDate
- Alternative view (when label was printed vs. packed)
- Still uses FalseMidnight setting

**Totals Table (Summary):**

- Aggregated from daily Serials by FM-adjusted date
- Daily totals already consolidated by FM "work day"
- Reports query Totals by FM-adjusted PdnDate

### 4.4 Sell-By / Expiration Date Calculations

**Relationship: FM affects shelf-life date assignment**

**Without FM:**

```
Kill (Process) Date: Sept 15, 8:00 PM
Sell-By Offset: 10 days
Shelf Life End: Sept 25
```

**With FM = 2 AM (FalseMidnightKill):**

```
Kill (Process) Date: Sept 15 (FM-adjusted)
FalseMidnightKill: 2 AM
Effective Kill Date for SellBy: Sept 15 (FM "work day")
Sell-By Offset: 10 days
Shelf Life End: Sept 25 (calculated from FM-adjusted date)
```

**Purpose:** Ensures shelf-life calculations align with actual production/kill timing

### 4.5 Label Printing & Date Codes

**Relationship: FM dates printed on labels**

Labels contain:

- **Print Date Code:** Derived from FM-adjusted PrintDate
- **Pack Date Code:** Derived from FM-adjusted PackDate
- **Kill Date Code:** Derived from FM-adjusted KillDate
- **Sell-By Date:** Calculated from kill date + offset

**Example Label:**

```
┌──────────────────────┐
│ PRODUCT 12301        │
│ Serial: 202409150001 │
│ Print Date: 09/15/24 │ ← FM-adjusted
│ Kill Date: 09/15/24  │ ← FM-adjusted
│ Sell By: 09/25/24    │ ← Based on FM kill date
│ Weight: 8.25 oz      │
│ Shift: 1             │
└──────────────────────┘
```

### 4.6 Operator Shift Assignment

**Relationship: FM boundaries align with shifts**

When Operator logs into label printing system:

```progress
FIND Operator WHERE Operator.Username = "jsmith"
IF CURRENT TIME IS BETWEEN Operator.ShiftStart AND Operator.ShiftEnd
    ASSIGN Serial.Shift = Operator.ShiftNumber
```

FM affects interpretation:

- Operator "night shift" = 10 PM to 6 AM
- With FM = 2 AM: All work by this operator assigned same FM "work day"
- Reports group by FM "day" and shift, not calendar day

### 4.7 Quality Control (QC) & Audit Trails

**Relationship: FM ensures quality records span same "work day"**

Quality audits for "Sept 15 shift 1 production":

```
WITHOUT FM:
  Audit date range: Sept 15 00:00 - Sept 15 23:59 (calendar day)
  Missing: Night shift cases (10 PM - 2 AM from real production)

WITH FM = 2 AM:
  Audit date range: Sept 15 02:00 - Sept 16 01:59 (FM day)
  Includes: All actual shift 1 night production
```

FM ensures QC records align with actual production windows.

### 4.8 Data Synchronization (Sent Flag)

**Relationship: FM ensures data exported in work-day batches**

Systems configured to export data by FM "work day":

```progress
FOR EACH Serial NO-LOCK
    WHERE Serial.PackDate = TODAY AND Serial.Sent = NO
```

With FM, "TODAY" means current FM "work day", not calendar day:

```
Calendar Date: Sept 16, 1:00 AM
FM = 2 AM
Actual FM "Work Day": Sept 15 (FM hasn't reset to Sept 16 yet)
Exports: All Sept 15 FM day cases NOT yet sent
```

Ensures complete by-FM-day synchronization without splits.

### 4.9 Scale & Equipment Assignment

**Relationship: FM groups production by equipment timeframe**

When analyzing scale performance:

```
Scale 1 Production Summary (Sept 15 with FM= 2 AM):
  Start: Sept 15, 2:00 AM
  End: Sept 16, 1:59 AM
  Total Cases: 523
  Avg Weight: 11.35 oz
  Uptime: 98.5%
```

FM ensures equipment metrics align with shift/operational boundaries.

### 4.10 Goal / Production Order Tracking

**Relationship: FM identifies which FM "work day" goal was executed**

Production Goals:

```
Goal ID: 5001
Target Product: 12301
Target Qty: 500 cases
Scheduled Date: Sept 15, 2024
```

With FM = 2 AM:

- Goal executed during Sept 15 FM work day (2 AM to 1:59 AM next day)
- Serial records linked to Goal show FM-adjusted dates
- Tracking aligns with when goal actually ran

---

## 5. False Midnight Algorithms

### 5.1 FM Before Noon (e.g., 2 AM)

**Logic (in pseudo-code):**

```
IF current_time >= FalseMidnight_time THEN
    work_day_date = calendar_date
ELSE
    work_day_date = calendar_date - 1
END IF
```

**Example (FM = 2:00 AM):**

```
Calendar: Sept 15, 6:00 AM
  6:00 AM >= 2:00 AM? YES
  → Work Day: Sept 15 ✓

Calendar: Sept 16, 1:30 AM
  1:30 AM >= 2:00 AM? NO
  → Work Day: Sept 15 (yesterday's FM day)

Calendar: Sept 16, 2:00 AM
  2:00 AM >= 2:00 AM? YES
  → Work Day: Sept 16 (FM boundary crossed)
```

### 5.2 FM After Noon (e.g., 10 PM)

**Logic (in pseudo-code):**

```
IF current_time >= FalseMidnight_time THEN
    work_day_date = calendar_date
ELSE
    work_day_date = calendar_date (but will switch at FM boundary)
END IF
```

**Example (FM = 10:00 PM):**

```
Calendar: Sept 15, 6:00 AM
  6:00 AM >= 10:00 PM? NO
  → Work Day: Sept 14 (was in yesterday's FM day)

Calendar: Sept 15, 6:00 PM
  6:00 PM >= 10:00 PM? NO
  → Work Day: Sept 14 (still in yesterday's FM day)

Calendar: Sept 15, 10:00 PM
  10:00 PM >= 10:00 PM? YES
  → Work Day: Sept 15 (FM boundary crossed)

Calendar: Sept 16, 8:00 AM
  8:00 AM >= 10:00 PM? NO
  → Work Day: Sept 15 (still in yesterday's FM day)
```

### 5.3 FM Disabled (24:00)

**Logic (in pseudo-code):**

```
IF FalseMidnight = '24:00' THEN
    work_day_date = calendar_date (no adjustment)
    Use standard midnight boundaries
END IF
```

**Effect:** Standard calendar days, no FM adjustment

---

## 6. Implementation Across SLC System

### 6.1 Label Printing (`lib/printlabel.p`)

**Purpose:** Print FM time code on labels (token substitution in label templates)

```abl
WHEN "FALSEMIDNIGHT" THEN
    chrReplacementText = STRING(gFalseMidNight, "HH:MM AM")
```

**Output on Label:**

```
FM: 02:00 AM
```

Labels can display the configured FM boundary for reference.

### 6.2 Serial Creation (`lib/createserial.p` and callers)

**Purpose:** Adjust PackDate and KillDate when creating Serial record.

The core logic lives in **`lib/createserial.p`** as procedure **`FalseMidnight`** (input: production time and date; output: adjusted PackDate and KillDate). It is invoked from:

- **`print/weigh.w`** when building the weigh session (vPackDate/vKillDate, then PackDateOffset/KillDateOffset applied).
- **`print/get-sellby-offset.p`** via internal procedure **`BuildFalseMidnightPDandKD`**, which calls `RUN falsemidnight IN gh_CreateSerial` with the same inputs/outputs.

**Process:**

1. Globals `gFalseMidnight` and `gFalseMidnightKill` are set at login from SiteConfigOption (see `lib/globalassigns.i`; session parm keys include `FalseMidNight` / `FalseMidNightKill`).
2. Values are already in seconds past midnight.
3. Compare production time to FM boundary: before noon (FM &lt; 43200): if time &lt; FM then date − 1; after noon: if time &gt; FM then date + 1.
4. KillDate is constrained so `poKillDate ≤ poPackDate`.
5. Callers then add `Modify.PackDateOffset` / `KillDateOffset` to get final Serial dates.

### 6.3 Production Reports (`reports/r-prodreps.w`)

**Purpose:** Filter Serials by FM "work day" using **Print Date** (not Pack Date).

The report uses `Serial.PrintDate` and `Serial.PrintTime` with FM-derived date and time bounds:

```progress
FIND SiteConfigOption WHERE OptionName = 'FalseMidnight'
IF OptionValue < '12:00' THEN  /* before noon */
    v-DateReportStart = fi-ReportDate
    v-DateReportEnd   = fi-ReportDate + 1
ELSE  /* noon or after */
    v-DateReportStart = fi-ReportDate - 1
    v-DateReportEnd   = fi-ReportDate
v-TimeReportStart = /* HH:MM from OptionValue converted to seconds */

FOR EACH Serial WHERE
    Serial.PrintDate >= v-DateReportStart AND Serial.PrintDate <= v-DateReportEnd
    AND (IF Serial.PrintDate = v-DateReportStart THEN Serial.PrintTime >= v-TimeReportStart ELSE TRUE)
    AND (IF Serial.PrintDate = v-DateReportEnd   THEN Serial.PrintTime <= v-TimeReportStart ELSE TRUE)
```

**Result:** Report shows FM-aligned "work day" totals by **Print Date**.

### 6.4 Serial to CSV Export (`reports/r-SerialsToCSV.w`)

**Purpose:** Export Serial records with FM-adjusted dates

CSV includes:

- PackDate (FM-adjusted)
- PrintDate (FM-adjusted)
- KillDate (FM-adjusted)
- DateCode (derived from PackDate)

Recipient receives FM-consolidated dates in CSV.

### 6.5 Data Modification (`sobjects/s-modify.w`)

**Purpose:** Handle FM when modifying existing serials

```progress
DEF VAR v-FalseMidnightSecs AS INTEGER NO-UNDO.
```

Tracks both:

- `gFalseMidnight` - FM time for pack dates
- `gFalseMidnightKill` - FM time for kill dates

Ensures modifications don't override FM adjustments already applied.

---

## 7. Configuration Examples

### Example 1: Manufacturing Facility with Night Shift (FM = 2 AM)

**Operations:**

```
Night Shift (10 PM - 6 AM) - Produces frozen products
Day Shift (6 AM - 2 PM) - Packaging and QC
Evening Shift (2 PM - 10 PM) - Cleanup and preparation
```

**Configuration:**

```
FalseMidnight: 02:00      (work day starts at 2 AM)
FalseMidnightKill: 02:00  (kill date aligns with work day)
```

**Effect:**

```
Night shift (10 PM - 6 AM):
  All work appears under same FM "work day"
  Shift 1 production Sept 15: 450 cases, 2 AM Sept 15 - 1:59 AM Sept 16

Day shift (6 AM - 2 PM):
  All work on same calendar day appears under same FM "work day"
  Shift 2 production Sept 15: 380 cases, all between 6 AM - 2 PM Sept 15
```

---

### Example 2: Retail Cold Storage (FM = 10 PM)

**Operations:**

```
Early Shift (6 AM - 2 PM) - Production
Late Shift (2 PM - 10 PM) - Packaging and cleanup
Night Shift (10 PM - 6 AM) - Maintenance, overnight processing
```

**Configuration:**

```
FalseMidnight: 22:00      (work day starts at 10 PM)
FalseMidnightKill: 22:00  (kill date aligns)
```

**Effect:**

```
Work Day Sept 15 (10 PM Sept 14 - 9:59 PM Sept 15):
  Early Shift: 6 AM - 2 PM Sept 15: 250 cases
  Late Shift: 2 PM - 10 PM Sept 15: 300 cases (starts on Sept 14 FM day?? - complex)
  Night Shift: 10 PM Sept 15: First cases of next FM day
```

**Note:** FM = 10 PM creates complex scenario where some of calendar Sept 15 belongs to FM Sept 14, rest to FM Sept 15

---

### Example 3: Disabled FM (24:00 - Standard Calendar)

**Configuration:**

```
FalseMidnight: 24:00      (standard calendar midnight)
FalseMidnightKill: 24:00
```

**Effect:**

```
No FM adjustment - pure calendar days
PackDate = calendar date when case produced
All alignment with midnight boundaries
No shift consolidation
```

---

## 8. Common Questions & Troubleshooting

### Q: Why is my morning shift production showing on yesterday's report?

**A:** This is likely FM working correctly. If FM = 2 AM and your shift starts at 6 AM, the 6 AM production belongs to today's FM "work day". Check:

- FM setting in SiteConfigOption
- Ensure shift times align with expected FM boundaries

### Q: The reports are showing dates that don't match what I expected

**A:** Verify:

1. What is FM configured to? (Check SiteConfigOption.OptionValue for 'FalseMidnight')
2. Is FM before or after noon? (determines calculation logic)
3. What date range are you selecting in report?
4. Are you filtering by PackDate vs. PrintDate?

### Q: Why do I have two FM settings (FalseMidnight and FalseMidnightKill)?

**A:** Different purposes:

- **FalseMidnight:** Aligns production/shift reporting to work days
- **FalseMidnightKill:** May use different boundary for shelf-life/expiration calculations
- Example: Production shifts at 2 AM, but shelf-life tracking uses 10 PM boundary

### Q: How do I verify FM is configured correctly?

**A:** Query the database:

```sql
SELECT OptionName, OptionValue
FROM SiteConfigOption
WHERE OptionName IN ('FalseMidnight', 'FalseMidnightKill');
```

Should return:

- FalseMidnight: value like "02:00", "22:00", or "24:00"
- FalseMidnightKill: value like "02:00", "22:00", or "24:00"

### Q: Can I change FM settings while production is running?

**Answer:** Technically yes, but **NOT RECOMMENDED**:

- ❌ May cause date inconsistencies
- ❌ Historical data uses old FM setting
- ❌ New serials use new FM setting → split data
- ✅ Change FM during off-hours or system downtime
- ✅ Document change in system changelog

---

## 9. Relationship Matrix

### Which Concepts Are Affected by FM?

| Concept        | Primary | Secondary | Affected Fields       | Impact Level |
| -------------- | ------- | --------- | --------------------- | ------------ |
| **Shift**      | ✅      | -         | Shift assignment      | **HIGH**     |
| **PackDate**   | ✅      | -         | Serial.PackDate       | **HIGH**     |
| **PrintDate**  | ✅      | -         | Serial.PrintDate      | **HIGH**     |
| **KillDate**   | ✅      | -         | Serial.KillDate       | **HIGH**     |
| **DateCode**   | ✅      | -         | Serial.DateCode       | **HIGH**     |
| **Reports**    | ✅      | -         | Date filtering        | **HIGH**     |
| **Labels**     | ✅      | -         | Printed dates         | **HIGH**     |
| **Sell-By**    | ✓       | ✅        | Expiration dates      | **MEDIUM**   |
| **Goals**      | ✓       | ✅        | Execution tracking    | **MEDIUM**   |
| **QC**         | ✓       | ✅        | Audit scope           | **MEDIUM**   |
| **Totals**     | ✓       | ✅        | Daily aggregates      | **MEDIUM**   |
| **Export/CSV** | ✓       | ✅        | Date values in output | **MEDIUM**   |
| **Operators**  | ✓       | ✅        | Shift boundary        | **LOW**      |
| **Products**   | -       | -         | None                  | **NONE**     |
| **Weights**    | -       | -         | None                  | **NONE**     |

---

## 10. Advanced Topics

### 10.1 FM with Distributed Systems

**Scenario:** Multiple plants with different shift patterns

```
Plant A (LA): FalseMidnight = 02:00 (night shift 10 PM - 2 AM)
Plant B (NY): FalseMidnight = 10:00 (night shift 2 AM - 6 AM)
Plant C (UK): FalseMidnight = 24:00 (standard, no FM)
```

**Challenge:** Consolidating data from multiple plants with different FM settings

**Solution:**

1. Store Plant ID in Serial records
2. When exporting, track Plant FM setting
3. Consolidate by Plant's local FM "work day"
4. Corporate dashboard converts to common "work day" if needed

### 10.2 FM with Real-Time Monitoring

**Scenario:** Live production dashboard needs to know current FM "work day"

**Logic:**

```
CurrentTime = NOW
CurrentDate = TODAY

FIND SiteConfigOption WHERE OptionName = 'FalseMidnight'
IF FMValue < '12:00' THEN
    IF CurrentTime >= FMValue THEN
        WorkDay = CurrentDate
    ELSE
        WorkDay = CurrentDate - 1
    END IF
ELSE
    IF CurrentTime >= FMValue THEN
        WorkDay = CurrentDate
    ELSE
        WorkDay = CurrentDate - 1
    END IF
END IF

DISPLAY WorkDay  /* Current FM "work day" */
```

### 10.3 FM Audit & Change Tracking

**Recommendation:** Log FM changes

```
Event: FM Configuration Changed
  Changed By: Administrator
  Changed At: 2024-03-11 15:30:00
  Old Value: 24:00 (disabled)
  New Value: 02:00 (2 AM)
  Reason: Facility activated night shift operations
  Impact: Future Serials will use FM adjustment
```

### 10.4 Migration: Enabling FM on Existing System

**Challenges:**

```
System has 6 months of production data with FM disabled (24:00)
Decision made: Enable FM = 2 AM going forward
```

**What happens:**

- ✅ New serials use FM = 2 AM adjustment
- ⚠️ Old serials from past 6 months have calendar dates (no FM)
- ⚠️ Reports mixing FM and non-FM dates create inconsistencies

**Solution:**

1. **Option A: Keep old data as-is**
   - New data: FM-adjusted from go-forward date
   - Old data: Remains calendar dates
   - Reports: Limited to post-FM-enable date in analysis

2. **Option B: Backfill old data (complex)**
   - Re-adjust historical PackDate/PrintDate/KillDate
   - Requires full audit trail
   - Risk of data inconsistency
   - **Not recommended** unless critical

**Best Practice:** Enable FM at system go-live before production begins

---



---

## 12. Configuration Checklist

### Implementing False Midnight

- [ ] Determine facility shift patterns (start/end times)
- [ ] Select FM time that aligns with natural shift boundary (e.g., 2 AM for 10 PM-6 AM shift)
- [ ] Access SiteConfigOption configuration menu
- [ ] Set FalseMidnight to HH:MM format (e.g., "02:00")
- [ ] Set FalseMidnightKill (same or different value)
- [ ] Test with new Serial records (verify PackDate/KillDate adjustment)
- [ ] Run test reports to verify consolidation
- [ ] Verify labels print correct dates
- [ ] Document FM setting and reason for facility records
- [ ] Train staff on new "work day" interpretation
- [ ] Monitor first week for anomalies

---

## 13. Summary

| Aspect                | Details                                                                |
| --------------------- | ---------------------------------------------------------------------- |
| **Purpose**           | Redefine "work day" to align with shift patterns                       |
| **Benefit**           | Consolidated reporting by actual shift work, not calendar days         |
| **Configuration**     | SiteConfigOption: FalseMidnight & FalseMidnightKill (HH:MM or "24:00") |
| **Scope**             | Affects dates in Serial records: PackDate, PrintDate, KillDate         |
| **Key Relationships** | Shifts, Reports, Labels, Sell-By dates, Data exports                   |
| **Default**           | "24:00" (disabled - standard calendar midnight)                        |
| **Complexity**        | Medium - requires understanding shift vs. calendar day distinction     |
| **Impact**            | HIGH - affects all date-based reporting and analysis                   |

---

