# Item Production Report: Complete Concept Guide


## 1. Overview

### What Is the Item Production Report?

The **Item Production Report** is a manufacturing analytics report that provides item-level production summaries for a configurable time period and shift(s). It answers the critical question: _"Which items were produced, how many units of each item were produced, and what are the weight statistics (labeled, net, average, give-away) for each item?"_

The report aggregates data from the **ItemSerial** table, grouping production by item code and manufacturing scale, allowing production supervisors and quality analysts to understand production volume and weight characteristics at the item level.

### Why It Exists (Business Value)

1. **Production Analysis**: Understand which items were produced in a given period and in what quantities
2. **Weight Compliance Verification**: Verify that items meet weight specifications (average, total, give-away)
3. **Yield Analysis**: Identify items with excessive give-away weights (negative margin between net and label weight)
4. **Shift-Specific Reporting**: Compare production across shifts or aggregate across all shifts
5. **Historical Tracking**: Query production by date packaged (pack date) or date printed/labeled (print date)
6. **Quality Metadata Export**: Export complete ItemSerial records as CSV for further analysis in Excel, databases, or BI tools

### Report Characteristics

| Attribute               | Value                                                                 |
| ----------------------- | --------------------------------------------------------------------- |
| **UI Entry Point**      | Main Menu → Reports → Item Production Report (s-repmenu.w button 257) |
| **Implementation File** | reports/itemprod.w                                                    |
| **Output Type**         | Dual output: formatted text report + CSV detailed export              |
| **Data Source**         | ItemSerial table                                                      |
| **Primary Filtering**   | Date range + date type (pack vs print) + shift selection              |
| **Aggregation Level**   | Item code (with sub-grouping by scale)                                |
| **Typical Users**       | Production supervisors, QA analysts, shift managers, plant operations |

---

## 2. Data Source: ItemSerial Table

### Core Table Definition

The Item Production Report queries the **ItemSerial** table, which tracks individual items (units) produced during manufacturing operations. Each ItemSerial record represents a single unit within a case/batch of production.

**Key Fields Used in Report:**

| Field         | Type      | Length | Purpose                                                               |
| ------------- | --------- | ------ | --------------------------------------------------------------------- |
| **ItemCode**  | Character | 5      | Item product code (primary grouping key)                              |
| **PackDate**  | Date      |        | Date item was packed/cased (one filtering option)                     |
| **PrintDate** | Date      |        | Date item was printed/labeled (alternative filtering option)          |
| **Shift**     | Integer   |        | Production shift (1, 2, 3, or 4 for multi-shift operations)           |
| **Scale**     | Integer   |        | Manufacturing scale/line identifier (sub-grouping for output)         |
| **LabelWgt**  | Decimal   | 9,2    | Labeled weight specification for this item (tare)                     |
| **NetWgt**    | Decimal   | 9,2    | Actual net weight of the item (product + tare)                        |
| **AddOrDel**  | Character | 1      | 'd' = deleted/canceled item, (blank) = normal production              |
| **SerialNum** | Character | 18     | Unique identifier for this item (composite key: SerialNum + AddOrDel) |

**For complete ItemSerial schema, see [SERIAL-CONCEPT-AND-USAGE.md](SERIAL-CONCEPT-AND-USAGE.md)**

### Filtering Logic

The report applies four filter conditions to ItemSerial:

```progress
FOR EACH ItemSerial NO-LOCK WHERE
    /* Date Range Filter (mutually exclusive) */
    (IF rsPackDate THEN
        (ItemSerial.PackDate >= fiStartDate AND
         ItemSerial.PackDate <= fiEndDate)
    ELSE
        (ItemSerial.PrintDate >= fiStartDate AND
         ItemSerial.PrintDate <= fiEndDate)
    )
    AND
    /* Shift Filter (mutually exclusive) */
    (IF rsAllShifts THEN
        TRUE  /* Include all shifts regardless of value */
    ELSE
        ItemSerial.Shift = INT(fi-Shift:screen-value)
    )
```

---

## 3. Report Parameters & User Interface

### Parameters

The report window (W-Win: 128.2 × 22.19 character grid) presents four main parameter selections:

#### A. Date Type Selection (Radio Set: `rsPackDate`)

Determines which date field is used for filtering:

- **PackDate** (YES) - Filter by the date the item was packed into its case/batch
  - Use Case: Production volume by packing/shifting operation
  - Example: "How much did we pack on Tuesday?"
- **PrintDate** (NO) - Filter by the date the item was printed/labeled
  - Use Case: Labeling/finishing operations, compliance with labeling timelines
  - Example: "How much did we label on Tuesday?"

**Implementation Detail**: The radio set is dynamically populated from localization resources:

```progress
v-RS-PackDate = get-Lang-Lbl(v-LangCode, "PrintLabels.Modify.PackDate") + ',' + 'YES,' +
                get-Lang-Lbl(v-LangCode, "Reports.ItemPdnRep.ReportDate") + ',' + 'NO'.
rsPackDate:RADIO-BUTTONS = v-RS-PackDate.
```

#### B. Date Range Selection (Fill-in Fields: `fiStartDate`, `fiEndDate`)

Specifies the date window for the report:

- **Start Date** (fiStartDate)
  - Default: Today's date (set at window initialization)
  - Input: Editable date field with spinner buttons (btnStartDateDown, btnStartDateUp)
  - Format: System date format (e.g., 12/11/02 in code, typically MM/DD/YY)

- **End Date** (fiEndDate)
  - Default: Today's date
  - Input: Editable date field with spinner buttons (btnEndDateDown, btnEndDateUp)
  - Format: System date format

**Spinner Button Behavior:**

- UP buttons add 1 day to the date
- DOWN buttons subtract 1 day from the date
- Allows rapid date navigation without manual typing

#### C. Shift Selection (Radio Set: `rsAllShifts`)

Determines shift filtering approach:

- **All Shifts** (YES) - Include items from all shifts in the report
  - Use Case: Plant-wide production summary
  - Filter: No shift constraint applied
- **Single Shift** (NO) - Include only items from selected shift
  - Use Case: Shift-specific analysis, comparative shift performance
  - Filter: ItemSerial.Shift = selected shift

**Implementation Detail**: When "Single Shift" is selected, the shift selection list (fi-Shift) becomes relevant:

```progress
rsAllShifts:RADIO-BUTTONS = get-Lang-Lbl(v-LangCode, "Reports.PdnRep.AllShifts") + ',' + 'YES,' +
                            '1' + ',' + 'NO'.
```

#### D. Shift Selection List (Selection List: `fi-Shift`)

When "Single Shift" mode is selected, this list allows shift selection:

- **Default Shifts**: Dynamically populated from ConfigOptionMaster
  - Configuration Option Name: `ValidShifts`
  - Default Value: '1|2|3|4' (pipe-delimited)
  - Can be customized per facility/config

- **Example Valid Values**: 1, 2, 3, 4 (typical for 4-shift operations)
- **Visible Rows**: 3 (shows 3 shifts at a time in selection list)
- **Selection Method**: Radio button list; select one shift to filter

**Initialization Code:**

```progress
FIND FIRST ConfigOptionMaster
    WHERE ConfigOptionMaster.OptionName = 'ValidShifts'
    NO-LOCK NO-ERROR.
IF AVAIL ConfigOptionMaster THEN
    fi-Shift:list-items = ConfigOptionMaster.AcceptableValues.
ELSE
    fi-Shift:list-items = '1|2|3|4'.
fi-Shift:screen-value = fi-Shift:entry(1).  /* Default to first shift */
```

### Output Buttons

The report provides four action buttons:

#### 1. **Run Report** (bu-Runrep)

- **Action**: Generates the report and displays formatted results in SELECT-1 list
- **Output**: Formatted text report to `temp/CTSItemProd.txt` + live display in UI
- **Handler**: `ON CHOOSE OF bu-Runrep: RUN GenerateReport.`

#### 2. **Print Report** (bu-PrintRep)

- **Action**: Prints the report (previously generated in Run Report) to system printer
- **Prerequisites**: Must have previously run the report (SELECT-1 list must contain items)
- **Behavior**: Uses ADM printing framework (`adecomm/_osprint.p`)
- **Handler**: Sends `temp/CTSItemProd.txt` to printer

#### 3. **Print To Label** (bu-PrintLab)

- **Action**: Would print item production results to label printers
- **Current Status**: **DISABLED/NOT IMPLEMENTED** (hidden in UI)
- **Comment in Code**: "this is not yet available: that pgm uses totals recs"
- **Related Procedure**: sobjects/s-prodtolab.w (disabled)
- **UI Status**: Hidden via `HIDE bu-PrintLab.` in local-initialize procedure

#### 4. **Quit** (bu-Quit)

- **Action**: Closes the report window
- **Handler**: `APPLY "CLOSE" TO THIS-PROCEDURE.`

---

## 4. Report Generation Logic (GenerateReport Procedure)

### 4.1 Initialization & Output Setup

The GenerateReport procedure performs the following initialization:

```progress
/* Capture user selections from UI */
ASSIGN rsPackDate         /* TRUE = PackDate filter, FALSE = PrintDate filter */
       fiStartDate        /* Start date from fill-in */
       fiEndDate          /* End date from fill-in */
       rsAllShifts        /* TRUE = all shifts, FALSE = single shift */
       fi-Shift           /* Selected shift number (if single shift mode) */

/* Localized labels for shift and scale headers */
v-char-shift = get-Lang-Lbl(v-LangCode,
    IF rsAllShifts THEN 'Reports.PdnRep.AllShifts' ELSE "Reports.ItemPdnRep.Shift")
v-char-scale = get-Lang-Lbl(v-LangCode, "Reports.ItemPdnRep.Scale")

/* Initialize accumulators for grand totals */
v-totcnt = 0    /* Grand total item count */
v-labwgt = 0    /* Grand total label weight */
v-netwgt = 0    /* Grand total net weight */
v-avgwgt = 0    /* Grand total average weight */
v-totgive = 0   /* Grand total give-away weight */

/* Build column headers for display */
v-line-1 = localized headers row 1 (Item, Total, Label, Net, Average, Total)
v-line-2 = localized headers row 2 (Code, Count, Wgt, Wgt, Wgt, Give-away)
v-Underline = "------- -------  ------------  ------------  --------..." (80 chars)

/* Clear output list and open output file */
SELECT-1:list-items = ""
OUTPUT TO "temp/CTSItemProd.txt"
```

### 4.2 Report Output Format (Formatted Iteration)

The first iteration through ItemSerial generates formatted human-readable output:

**Output Flow:**

1. **Report Header**
   - Current date/time
   - Report title (localized)
   - Column header rows
   - Underline separator

2. **Iteration Over ItemSerial** (BREAK BY Scale, ItemCode)
   - **First-of(Scale)**: Print scale/shift header
     - Shift number (if single shift) or empty (if all shifts)
     - Scale number
     - Sub-underline
     - Add lines to SELECT-1 display list

   - **Last-of(ItemCode)**: Print item summary line
     - Item Code (5-digit formatted)
     - Item Count (with accounting minus if negative)
     - Item Label Weight (formatted with thousand separators)
     - Item Net Weight (formatted with thousand separators)
     - Item Average Weight (NetWgt / Count, or 0 if count = 0)
     - Item Give-away Weight (NetWgt - LabelWgt)
     - Add line to SELECT-1 display list

   - **Last-of(Scale)**: Print scale subtotal
     - "Total" label
     - Scale Count, Label Weight, Net Weight, Average Weight, Give-away Weight
     - Add line to SELECT-1 display list

3. **File Output**
   - Each line is sent to both:
     - Console (PUT UNFORMATTED): displayed in report window
     - SELECT-1 list (add-last): added to on-screen list for user viewing

### 4.3 Aggregation Rules

#### Item Count Aggregation

Items are counted with special handling for cancellations:

```progress
IF ItemSerial.AddOrDel = 'd' THEN
    v-ItemTotcnt = v-ItemTotcnt - 1  /* Canceled = -1 count */
ELSE
    v-ItemTotcnt = v-ItemTotcnt + 1  /* Normal = +1 count */
```

**Business Logic**: Canceled items represent products that were produced but later deleted from the production record (e.g., defects removed from inventory). The negative count shows net production after cancellations.

#### Weight Aggregation

All weights are accumulated by dropping the AddOrDel filter (all items, including canceled items, contribute to weight):

```progress
v-ItemLabwgt = v-ItemLabwgt + ItemSerial.LabelWgt  /* Labeled weight total */
v-ItemNetwgt = v-ItemNetwgt + ItemSerial.NetWgt    /* Net weight total */
```

#### Average Weight Calculation

Average weight is calculated only when count > 0:

```progress
v-avgwgt = IF v-ItemTotCnt > 0
           THEN (v-ItemNetwgt / v-ItemTotcnt)
           ELSE 0
```

**Implication**: If an item has equal number of additions and deletions (net count = 0), average weight is set to 0 to avoid division errors.

#### Give-away Weight Calculation

Give-away weight represents the weight difference between net and label weight (allowable variance):

```progress
v-totgive = v-ItemNetWgt - v-ItemLabWgt
```

**Business Context**:

- Positive value: Product weight exceeds label weight (favorable)
- Negative value: Product weight is less than label weight (shortfall)
- Expected range: -0.5 to +0.5 oz (small tolerance)
- Use case: Quality assurance verifies give-away within specification

### 4.4 Data Export (CSV Iteration)

After formatted report generation completes, the procedure runs a second iteration to export full ItemSerial records as CSV:

```progress
/* Build column headings from metaschema */
FOR EACH _file NO-LOCK
    WHERE _file-name = 'serial':
    FOR EACH _field OF _file NO-LOCK
        BY _order:
        vColumnHeadings = vColumnHeadings + _field._field-name + ','
    END
END

/* Output CSV with headings */
PUT UNFORMATTED vColumnHeadings SKIP

/* Export each ItemSerial record */
FOR EACH ItemSerial NO-LOCK WHERE (same date/shift filters):
    EXPORT DELIMITER ',' ItemSerial
END

OUTPUT CLOSE
```

**CSV Destination**: `temp/Items-[StartDate]-to-[EndDate]-shift-[ShiftNum].csv`

**Example Filename**: `temp/Items-01-15-24-to-01-20-24-shift-all.csv`

**Content**: All ItemSerial fields in column order, one record per line, useful for Excel analysis or data warehouse import.

---

## 5. Relationships to Other Concepts

### ItemSerial Table (Primary Data Source)

- **Relationship**: Report queries ItemSerial directly
- **Link Fields**: ItemCode, PackDate, PrintDate, Shift, Scale, LabelWgt, NetWgt
- **Cardinality**: One ItemSerial record → one line in report detail
- **For Details**: See [SERIAL-CONCEPT-AND-USAGE.md](SERIAL-CONCEPT-AND-USAGE.md) § 3

### Product Table (Item Master)

- **Relationship**: ItemCode in ItemSerial references Product
- **Link Fields**: ItemCode (in both tables)
- **Data Used**: Item code lookup, product properties (not directly in report but available via join)
- **When Relevant**: Understanding what items are (product names, categories, specifications)
- **For Details**: See [ITEM-CONCEPT-AND-USAGE.md](ITEM-CONCEPT-AND-USAGE.md)

### Serial Table (Case/Batch Master)

- **Relationship**: Each ItemSerial belongs to a Serial (case)
- **Link Fields**: SerialNum (first part of ItemSerial's composite key)
- **Data Impact**: Serial aggregates items from ItemSerial
- **When Relevant**: If need to trace back to which case/batch each item belongs
- **For Details**: See [SERIAL-CONCEPT-AND-USAGE.md](SERIAL-CONCEPT-AND-USAGE.md)

### ConfigOptionMaster Table (System Configuration)

- **Relationship**: Shift list loaded from ConfigOptionMaster
- **Configuration Option**: 'ValidShifts'
- **Usage**: `fi-Shift:list-items` populated from `ConfigOptionMaster.AcceptableValues`
- **Example**: '1|2|3|4' (pipe-delimited shift codes)
- **Impact**: Report must reflect valid shifts for the facility

### Goals Table (Production Targets)

- **Relationship**: Indirect - Goals sets expected production; report shows actual
- **Link Fields**: Product → Goal relationship
- **Use Case**: Compare report item counts against production goals
- **When Relevant**: Goal achievement analysis, variance reporting
- **For Details**: See [GOALS-CONCEPT-AND-USAGE.md](GOALS-CONCEPT-AND-USAGE.md) (if documented)

### Totals Table (Aggregated Metrics)

- **Relationship**: Parallel aggregation - Totals pre-calculates some metrics
- **Note**: "Print To Label" button is disabled because it uses Totals (not yet available)
- **Future**: May integrate Totals for faster report generation on large datasets

### Modify Table (Production Runtime Adjustments)

- **Relationship**: Modifications to product specifications during production
- **Link Fields**: ItemSerial inherits final weights after Modify operations applied
- **Data Impact**: LabelWgt, NetWgt in ItemSerial reflect post-Modify values
- **When Relevant**: Understanding what weight specifications were actually in use during production

---

## 6. Business Rules

### Creation & Filtering Rules

**Rule 6.1: Date Type Selection Mutual Exclusivity**

- Only ONE of PackDate or PrintDate is used for filtering per report run
- User selects via radio button: PackDate (YES) or PrintDate (NO)
- Selection determines which date field ItemSerial is filtered on
- Cannot filter by both dates simultaneously

**Rule 6.2: Shift Filtering Mutual Exclusivity**

- Only ONE of "All Shifts" or "Single Shift" is active per report run
- If All Shifts = YES: rsAllShifts filter condition is TRUE (no constraint)
- If All Shifts = NO: Filter constrains to ItemSerial.Shift = selected shift value
- Single shift selection has no effect if All Shifts is selected

**Rule 6.3: Valid Date Range**

- Start Date must be <= End Date (not enforced by UI; user responsibility)
- If Start Date > End Date: report returns zero items (no records match)
- Date format follows system locale (typically MM/DD/YY in code, system-dependent)

**Rule 6.4: Active Shift Availability**

- Shifts included in fi-Shift selection list come from ConfigOptionMaster
- ConfigOptionMaster.OptionName = 'ValidShifts'
- If ConfigOptionMaster record not found: defaults to '1|2|3|4'
- Facilities can customize valid shifts per location

### Aggregation & Calculation Rules

**Rule 6.5: Item Count Includes Cancellations as Negative**

- Items with AddOrDel = 'd' (deleted/canceled) count as -1
- Items with AddOrDel = NULL/blank count as +1
- Final item count reflects net production (additions minus cancellations)
- Example: 100 items added + 3 items canceled = count of 97

**Rule 6.6: Weight Totals Include All Items Regardless of Cancellation**

- LabelWgt and NetWgt accumulators include both active and canceled items
- Weight is attributed at the time of production, not revoked on cancellation
- This allows quality analysis of canceled items (e.g., "why were these canceled?")

**Rule 6.7: Average Weight Calculated Only If Count > 0**

- Average Weight = NetWgt / Count
- If Count = 0: Average Weight = 0 (no division by zero)
- Average Weight represents typical weight per item for the item code
- Used for specification verification and weight compliance

**Rule 6.8: Give-away Weight = Net Weight - Label Weight**

- Positive value: Product heavier than labeled (weight allowance used)
- Negative value: Product lighter than labeled (shortfall)
- Typical specification: ±0.5 oz tolerance
- Cumulative across all items in the report period

**Rule 6.9: Grouping and Breaking**

- Primary break: Scale (manufacturing line/scale ID)
  - New scale = print scale header
  - Last scale = print scale subtotal
- Secondary break: ItemCode (product item code)
  - Last item code within a scale = print item line
  - Reset item counters
- Grand total: Last of all iterations = print overall totals

**Rule 6.10: Report Output Separate from Data Export**

- Report generation produces TWO outputs:
  1. Formatted text report: `temp/CTSItemProd.txt` (human-readable)
  2. CSV data export: `temp/Items-[dates]-shift-[num].csv` (data analysis)
- Both use same ItemSerial filters
- Both run in single GenerateReport execution for consistency

### Weight Compliance Rules

**Rule 6.11: Label Weight Represents Tare**

- LabelWgt is the labeled tare (container/packaging weight)
- Used to calculate net product weight
- Typically fixed per item code
- Should match product specifications in Product table

**Rule 6.12: Net Weight Includes Raw Material Plus Tare**

- NetWgt = product weight + tare weight (as placed on scale)
- This is the measured weight recorded by the weighing system
- LabelWgt should be subtracted to get true product weight: ProductWgt = NetWgt - LabelWgt

**Rule 6.13: Give-away Weight for Compliance Monitoring**

- Calculated as: NetWgt - LabelWgt (product weight after removing tare)
- Positive give-away: expected, indicates weight allowance buffer
- Negative give-away: shortfall, may indicate filling/sealing issues
- Used for process control and quality assurance

### Cancellation & Production Record Rules

**Rule 6.14: Canceled Item Handling**

- AddOrDel = 'd' indicates item was deleted from production
- Reasons: defect found, weight out of spec, intentional batch rejection
- Cancellation decreases net count but does NOT remove weight records
- Allows analysis: "We had 100 items produced but 3 were rejected"

**Rule 6.15: Report Reflects Actual Production plus Cancellations**

- Positive count = net items shipped/completed
- Item count visibility = net of cancellations
- Weight totals = all items produced during period (accounting only for cancellations in count)

---

## 7. Real-World Usage Scenarios

### Scenario 1: Daily Production Review (Shift Manager)

**Context**: Shift end-of-day production meeting

**User Action**:

1. Opens Item Production Report
2. Selects:
   - PackDate: YES (items packed during shift)
   - Start Date: today
   - End Date: today
   - All Shifts: NO, select Shift 1
3. Clicks "Run Report"

**Report Output Example**:

```
01/15/24  14:22:33                   CTS Item Production Reports

ITEM      TOTAL  LABEL             NET               AVERAGE           TOTAL
CODE      COUNT  WGT               WGT               WGT               GIVE-AWAY

------- -------  ------------  ------------  ------------  ----------------

Shift : 1 Scale : 1
--------------------
12301        487  3,490.22      3,987.65      8.18          497.43
12302        512  4,096.00      4,610.33      9.00          514.33
Total        999  7,586.22      8,597.98      8.61          1,011.76

Shift : 1 Scale : 2
--------------------
12301        445  3,185.00      3,630.45      8.16          445.45
12305        203  1,624.00      1,852.35      9.13          228.35
Total        648  4,809.00      5,482.80      8.46          673.80

GRAND TOTAL  1,647  12,395.22 14,080.78 8.55 1,685.56
```

**Business Insights**:

- Shift 1 produced 1,647 items across 2 scales
- Scale 1 had 2 items; Scale 2 had 2 items
- Average weight across shift: 8.55 oz per item (should be ~8.0 oz based on spec)
- Total give-away: 1,685.56 oz across entire shift (slightly high, investigate process)
- Item code 12302 had highest volume (512 items)

**Actions Taken**:

- QA verifies weight averages are within specification (8.0 ± 0.5 oz)
- Identifies Scale 1 running slightly heavy (8.61 oz avg vs 8.55 shift avg)
- Flags give-away weight for process optimization review

---

### Scenario 2: Multi-Shift Production Comparison

**Context**: Plant manager weekly analysis

**User Action**:

1. Opens Item Production Report
2. Selects:
   - PackDate: YES
   - Start Date: Monday (01/08/24)
   - End Date: Friday (01/12/24)
   - All Shifts: YES (all shifts combined)
   - Run Report

**Report Output** (summarized):

```
Overall Week Production Summary:

Item 12301: 2,450 total units, 18,000 oz net, 7.35 oz avg
Item 12302: 2,890 total units, 26,010 oz net, 9.00 oz avg
Item 12305: 1,200 total units, 10,800 oz net, 9.00 oz avg
Goal This Week: 6,540 items

Grand Total: 6,540 items (100% of goal achieved!)
```

**Business Insights**:

- Weekly production target met exactly (6,540 items)
- Item 12302 highest volume but requires attention (9.00 oz avg, specification 8.0 ± 0.5 oz)
- Item 12301 running light (7.35 oz, below specification minimum of 7.5 oz)
- Item 12305 performs within spec (9.00 oz, acceptable for product)

**Actions Taken**:

- Recommend scale calibration for line running item 12301 (underweight)
- Review filler settings for item 12302 (overweight)
- Schedule preventive maintenance if pattern continues

---

### Scenario 3: Yield Analysis - Identifying Problem Items

**Context**: Quality engineer investigating defect rate

**User Action**:

1. Opens Item Production Report (three times with different filters)
2. First Run: PrintDate YES, all shifts, last 2 weeks
   - Result: 8,542 items total
3. Second Run: Same as above but adds focus to item with highest cancellation
4. Third Run: Compare by shift to find if problem is shift-specific

**Report with Cancellations** (items where count is negative):

```
Item 15401: 434 total count (original 468 produced, 34 canceled = 434 net)
           - Indicates 7.3% rejection rate

Item 15402: 612 total count (original 615 produced, 3 canceled = 612 net)
           - Indicates 0.5% rejection rate (normal)
```

**Analysis**:

- Item 15401 has 7.3% cancellation rate (PROBLEM - should be < 2%)
- Item 15402 operating normally
- Next step: Check Shift 2 separately to see if problem is shift-specific

**Root Cause Investigation**:

- Query ItemSerial for item 15401 with AddOrDel = 'd'
- Examine weight variance on canceled items
- Check if shift-specific (all Shift 2? all Shift 1?)
- Correlate with weight calibration/adjustment records in Modify table

---

### Scenario 4: Compliance & Traceability - Exporting Data

**Context**: Food safety auditor requires documentation

**User Action**:

1. Opens Item Production Report
2. Selects:
   - PackDate: YES
   - Start Date: 01/01/24
   - End Date: 01/31/24
   - All Shifts: YES
3. Clicks "Run Report"
4. Once report displays, clicks "Print Report" or exports CSV

**CSV Export Created**: `temp/Items-01-01-24-to-01-31-24-shift-all.csv`

**CSV Contents** (sample headers):

```
SerialNum,ItemCode,PackDate,PrintDate,Shift,Scale,LabelWgt,NetWgt,AddOrDel,...
[all ItemSerial fields follow]
```

**Use Cases**:

- Import into Excel for detailed analysis
- Send to regulatory/compliance team
- Load into data warehouse for historical tracking
- Create formatted audit report with formulas, charts, etc.

**Audit Requirements Met**:

- Complete list of items produced with weights
- Traceability from item serial number to case serial number
- Date and shift information for production tracking
- Tare and net weight for compliance verification

---

## 8. Common Queries & Data Interpretation

### Query 1: Identify Items Below Weight Specification

**Question**: "Which items in this week's production are running light (below spec)?"

**Answer from Report**:

1. Run report with PackDate range covering the week
2. Look at "AVERAGE" column for each item
3. Compare to item specification (typically 8.0 ± 0.5 oz for most items)
4. Flag items with average < 7.5 oz

**Example**:

```
Item 12301: Average 7.35 oz → BELOW SPEC (minimum 7.5 oz)
Item 12302: Average 9.00 oz → WITHIN SPEC (7.5 - 8.5 oz range)
```

**Root Cause Indicators**:

- Underfill on filling equipment
- Weight calibration drift on scale
- Tare weight inaccuracy (LabelWgt incorrect)

---

### Query 2: High Give-away Weight Analysis

**Question**: "Why are we giving away too much product (give-away weight too high)?"

**Answer from Report**:

1. Look at "TOTAL GIVE-AWAY" column
2. Calculate average: Total Give-away / Count = per-item give-away
3. Compare to specification (typically ±0.5 oz)

**Example**:

```
Item 12302: Total Give-away = 514.33 oz, Count = 512 items
            Per-item give-away = 514.33 / 512 = 1.004 oz PER ITEM
            Specification: ±0.5 oz max

            STATUS: EXCESSIVE - Each item is ~1 oz heavier than labeled

Cost Impact: 1 oz × 512 items = 512 oz per day
            Over month: 512 oz/day × 20 days = 10,240 oz (~640 lbs)
            At $3/lb: $1,920/month loss due to overfilling
```

**Root Cause Indicators**:

- Filling equipment calibration (overfill)
- Tare weight specification incorrect (LabelWgt too low)
- Operator overfilling manually

---

### Query 3: Track Item Cancellations by Shift

**Question**: "Which shift has the highest rejection/cancellation rate?"

**Interpretation from Report**:

1. Run report three times: once for each shift (Shift 1, 2, 3, 4)
2. Calculate cancellation rate: (Original Produced - Net Count) / Original Produced × 100%
3. Compare across shifts

**Method**:

- Run with All Shifts = NO, select Shift 1 → get count X
- Run with All Shifts = NO, select Shift 2 → get count Y
- Run with All Shifts = YES → get total count Z
- If Z is less than X+Y+..., the difference indicates cancellations

**More Direct Method**:

- Export CSV for each shift
- Sort by AddOrDel = 'd'
- Count canceled items per shift
- Calculate percentage

**Example**:

```
Shift 1: 1,200 items produced, 18 canceled = 1.5% rejection rate (GOOD)
Shift 2: 1,100 items produced, 78 canceled = 7.1% rejection rate (PROBLEMATIC)
Shift 3: 980 items produced, 5 canceled = 0.5% rejection rate (EXCELLENT)
Shift 4: 890 items produced, 44 canceled = 4.9% rejection rate (NEEDS REVIEW)

ACTION: Investigate Shift 2 - high rejection rate suggests equipment, material, or process issue
```

---

### Query 4: Product-Specific Production Trends

**Question**: "How is item 15401 performing over time (trend analysis)?"

**Approach**:

1. Run report multiple times with same item focus but different date ranges
2. Maintain results in spreadsheet or BI tool
3. Plot Average Weight, Total Count, Give-away over time

**Example Data**:

```
Week      Count    Avg Wgt    Give-away/Item    Notes
01/01-07  512      8.15 oz    0.35 oz           Normal
01/08-14  498      7.85 oz    0.15 oz           Within spec
01/15-21  445      7.35 oz    -0.15 oz          Below spec, shortfall
01/22-28  460      7.42 oz    -0.08 oz          Still low, check filler
```

**Trend Identification**:

- Item 15401 showing drift toward underweight over past 3 weeks
- Recommends scale calibration or filler adjustment
- If pattern continues, quality may be compromised

---

## 9. Integration Points

### 1. Link to Batch/Case Information

**Question**: "Which case(s) contain item serial #W2024010115?"

**Integration Path**:
ItemSerial.SerialNum → Serial table

- SerialNum is first part of ItemSerial's composite key
- Join: ItemSerial.SerialNum = Serial.SerialNum
- Get: case-level data (case weight, packed date, batch info)

**Use Case**: Trace single item back to its case for quality investigation

---

### 2. Link to Product Master Data

**Question**: "What is the product name and specification for item code 12301?"

**Integration Path**:
ItemSerial.ItemCode → Product / Item table

- Link: ItemSerial.ItemCode = Product.ItemCode
- Get: product name, category, specification, target weight

**Use Case**: Display item name instead of code in reports, verify weight against spec

---

### 3. Link to Production Goals

**Question**: "Did we meet production goals for items in this report?"

**Integration Path**:

1. Run Item Production Report to get item counts
2. Cross-reference with Goals table
   - Goals.ProductID = Item.ItemCode (conceptual link)
   - Goals.TargetQty = expected count
3. Calculate: Actual / Target = %Achievement

**Use Case**: "We produced 2,450 units of item 12301 this week against a goal of 2,400 = 102% achievement"

---

### 4. Link to Scale/Device Configuration

**Question**: "Which scale was running item 15401, and what are its specifications?"

**Integration Path**:
ItemSerial.Scale → Device table

- Link: ItemSerial.Scale = Device.DeviceID
- Get: device name, type, calibration_date, location

**Use Case**: Correlate scale-specific weight drift with device maintenance schedule

---

### 5. Export to Quality Management System

**Question**: "How do we send this production data to our QMS for compliance tracking?"

**Integration Path**:

1. Generate report and export CSV
2. CSV contains all ItemSerial fields with complete traceability data
3. Import CSV into QMS database or Excel template
4. QMS tracks weight compliance, generates certificates, maintains audit trail

**Typical Fields Exported**: SerialNum, ItemCode, PackDate, Shift, Scale, LabelWgt, NetWgt, AddOrDel, etc.

---

## 10. Performance Considerations

### Dataset Size Impact

**Typical Scenario**:

- Production facility: 4 shifts, 8 hours/shift, 2 items per scale, 2 scales
- Throughput: ~1,500 items/day
- ItemSerial records: 1,500 items/day × 365 days = 547,500 records/year

**Report Filtering Impact**:

- Single day report (1 shift): filters to ~375 records (0.07% of annual data) → instant execution
- Single month report (all shifts): filters to ~45,000 records (8% of annual data) → <1 second execution
- Annual report (all shifts): filters to 547,500 records (100%) execution → 5-10 second processing

**Optimization Opportunities**:

1. Index on ItemSerial(PackDate) or ItemSerial(PrintDate) - improves date range filtering
2. Index on ItemSerial(Shift) - improves shift filtering
3. Consider archiving old ItemSerial data (older than 1 year) to separate table

### Memory & Output Handling

**Report Size**:

- Formatted text report: ~50 KB per 1,000 items
- CSV export: ~100 KB per 1,000 items (includes all fields)
- SELECT-1 display list: ~1 MB per 10,000 line items

**Considerations**:

- Large multi-month reports may slow UI responsiveness
- CSV export filename includes date range for clarity: `Items-01-01-24-to-01-31-24-shift-all.csv`
- Recommend limiting reports to 30-day windows for best performance

---

## 11. Troubleshooting

### Issue: Report Shows No Data

**Possible Causes**:

1. **Date range mismatch**: Start Date > End Date, or dates with no production
2. **New facility setup**: No ItemSerial records yet for selected period
3. **Shift filter**: Selected shift had no production on those dates
4. **Filter persistence**: Previously selected filters still active

**Debugging Steps**:

1. Verify Start Date <= End Date
2. Run report with broadest filter: All Shifts, entire month, any date type
3. Check that ItemSerial table has data (query database directly if needed)
4. Try different date ranges to isolate issue

---

### Issue: Weights Display Incorrectly

**Possible Causes**:

1. **Localization/format issue**: Decimal separator (. vs , ) might be locale-dependent
2. **Data type mismatch**: LabelWgt or NetWgt not properly numeric in database
3. **Null values**: Some items have null weights (data entry error)

**Debugging Steps**:

1. Export CSV and open in Excel to verify numeric formatting
2. Query ItemSerial directly for null weight values
3. Check database type definitions: LabelWgt and NetWgt should be DECIMAL(9,2)

---

### Issue: CSV Export File Not Found

**Possible Causes**:

1. **temp/ folder does not exist**: Report cannot write to non-existent directory
2. **Permission denied**: User lacks write access to temp folder
3. **File locked**: Previous CSV file still open in Excel

**Debugging Steps**:

1. Verify temp/ directory exists (relative to application directory)
2. Check user permissions on temp/ folder
3. Close Excel or temp/ files before re-running report

---

### Issue: "Print To Label" Button Disabled

**Status**: This is **intentional and expected** - not a bug

**Reason**: Button calls sobjects/s-prodtolab.w which relies on Totals table for aggregated metrics. Totals table might not be populated or might use different calculation logic than ItemSerial direct queries. Feature is disabled until Totals integration is complete.

**Workaround**: Use "Run Report" or "Print Report" buttons instead for standard text output.

---
