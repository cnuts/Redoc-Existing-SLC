## 1. Overview

### What Is the Serials to CSV Report?

The **Serials to CSV Report** is a detailed production case/box-level data export that converts all individual Serial records into comma-separated values (CSV) format. It answers critical operational questions:

- _"What individual cases/boxes were produced during a date range?"_
- _"What were the exact weight measurements for each case?"_
- _"Which cases were added vs. cancelled/deleted?"_
- _"What is the complete traceability data for quality/regulatory compliance?"_

The report queries the **Serial** table with configurable date and shift parameters, then exports all matching records as CSV with both a formatted summary display AND a raw data export file. This allows production personnel, quality analysts, and system administrators to:

1. Export complete case-level production history for analysis
2. Create compliance documentation with full traceability
3. Generate detailed weight specification reports by case
4. Perform quality audits on individual units
5. Integrate production data with external systems (Excel, databases, BI tools)

### Why It Exists (Business Value)

1. **Complete Traceability**: Every individual case/box has a unique SerialNum with full tracking data
2. **Quality Compliance**: Export weight data for quality specifications and regulatory compliance
3. **Detailed Analysis**: Support detailed analysis at the case level (not just summaries)
4. **Data Integration**: Enable export to Excel, databases, and third-party systems
5. **Historical Record**: Create timestamped records of all production cases for audit trail
6. **Problem Investigation**: Trace specific cases when quality issues arise
7. **Customer Communication**: Provide detailed case information for customer inquiries
8. **Add/Delete Tracking**: Track which cases were cancelled or rejected
9. **Shift-based Reporting**: Compare production by shift with complete detail
10. **Data Validation**: Verify data integrity by reviewing complete serial records

### Report Characteristics

| Attribute                | Value                                                                        |
| ------------------------ | ---------------------------------------------------------------------------- |
| **UI Entry Point**       | Main Menu → Reports → Serials to CSV                                         |
| **Implementation File**  | `reports/r-SerialsToCSV.w`                                                   |
| **Report Type**          | Transaction-level data export (configurable by date and shift)               |
| **Data Source**          | Serial table (with optional filtering)                                       |
| **Sort Order**           | By Scale, then by ProductCode (as displayed); by ProductCode (as exported)   |
| **Output Formats**       | CSV file export + formatted text summary display                             |
| **Display Format**       | Selection list with summary statistics and totals                            |
| **CSV Export Format**    | Comma-delimited file with all Serial table fields                            |
| **Export File Location** | `temp/serials-[StartDate]-to-[EndDate]-shift-[ShiftNum].csv`                 |
| **Pagination**           | Full dataset (no page limits)                                                |
| **Typical Users**        | Quality analysts, production supervisors, data analysts, compliance officers |
| **Data Volume**          | Hundreds to tens of thousands of serial records per export                   |

---

## 2. Data Source: Serial Table

### Core Table Definition

The **Serial** table contains the master record for every individual case/box produced in the manufacturing system. Each Serial record represents a unique production unit with complete specifications and traceability data.

#### Key Fields Used in Export

| Field                          | Type      | Size | Purpose                                                |
| ------------------------------ | --------- | ---- | ------------------------------------------------------ |
| **SerialNum**                  | Character | 12   | Unique 12-digit serial number (case/box identifier)    |
| **AddOrDel**                   | Character | 1    | 'A' = Added, 'D' = Deleted/Cancelled                   |
| **PrintDate**                  | Date      | 4    | Date the label was printed                             |
| **PrintTime**                  | Integer   | 4    | Time label was printed (seconds since midnight)        |
| **PackDate**                   | Date      | 4    | Date the case/box was packed/produced                  |
| **KillDate**                   | Date      | 4    | Date the product was actually killed/processed         |
| **Shift**                      | Integer   | 2    | Shift number (1, 2, 3, etc.)                           |
| **Operator**                   | Character | 10   | Username/ID of operator who printed label              |
| **Sent**                       | Logical   | 1    | Whether record has been sent to external system        |
| **ScaleNumber**                | Integer   | 2    | Scale/equipment identifier where case was produced     |
| **SellByOffset**               | Integer   | 2    | Days offset from KillDate for sell-by dating           |
| **VarOffset**                  | Integer   | 2    | Variable offset from today                             |
| **WgtUnits**                   | Character | 2    | Weight units: 'LB' or 'KG'                             |
| **LabelWgt**                   | Decimal   | 7,3  | Label weight (packaging tare weight)                   |
| **NetWgt**                     | Decimal   | 7,3  | Net weight (actual product weight)                     |
| **BoxTare**                    | Decimal   | 7,3  | Tare weight of box/packaging                           |
| **ExtraTare**                  | Decimal   | 7,3  | Additional tare (moisture, etc.)                       |
| **ProductCode**                | Integer   | 5    | Product code (00001-99999)                             |
| **Lot**                        | Character | 10   | Lot number (alpha-numeric)                             |
| **GoalID**                     | Integer   | 4    | Reference to production goal                           |
| **Reprinted**                  | Logical   | 1    | Whether label has been reprinted                       |
| **DateCode**                   | Character | 10   | Formatted date code (YYYY-MM-DD format)                |
| _[Plus ~30 additional fields]_ |           |      | Details on labeling, dating, customer, billing, status |

---

## 3. Report Parameters & Selection Criteria

### Parameter Configuration UI

The report presents a parameter selection window (based on r-SerialsToCSV.w) with the following controls:

**Date Range Parameters:**

```
Start Date [__________] Up / Down buttons
End Date   [__________] Up / Down buttons
```

- Default Start Date: Today - 80 days (recent production)
- Default End Date: Today
- Format: MM/DD/YYYY
- Users can manually adjust dates or use Up/Down buttons to increment/decrement

**Date Type Selection:**

```
○ Pack Date  ○ Print Date
```

**Shift Selection:**

```
○ All Shifts  ○ This Shift [Dropdown: 1, 2, 3, ...]
```

**Button Actions:**

- **Run Report**: Generate CSV export and display summary
- **Print Report**: Print formatted summary to printer
- **Print To Label**: Generate label printing queue (future feature)
- **Quit**: Exit the report window

### Filtering Logic

**Date Range Filtering:**

```progress
FOR EACH Serial NO-LOCK
    WHERE (IF rsPackDate THEN
             (Serial.PackDate >= StartDate AND Serial.PackDate <= EndDate)
           ELSE
             (Serial.PrintDate >= StartDate AND Serial.PrintDate <= EndDate))
```

**Shift Filtering:**

```progress
    AND (IF rsAllShifts THEN
           TRUE
         ELSE
           Serial.Shift = SelectedShift)
```

**Reprinted Filtering:**

```progress
    AND NOT Serial.Reprinted
```

- Excludes reprinted/duplicate labels from the report
- Only includes original and valid serial records

---

## 4. CSV Generation Process

### 4.1 Procedure: ipCreateCSV

This is the core CSV export procedure that creates the raw data file.

**Key Steps:**

1. **Build Output Filename:**

   ```
   temp\serials-[StartDate]-to-[EndDate]-shift-[ShiftNum].csv
   ```

   Example: `temp\serials-01-15-24-to-01-31-24-shift-all.csv`

2. **Query Data Dictionary:**
   - Scans the `_file` and `_field` metadata tables
   - Extracts field names for the Serial table in proper sequence
   - Builds column header row dynamically

3. **Export Column Headers:**
   - First line of CSV contains all field names from Serial table
   - Order matches database schema order (by ORDER field in .df)
   - Comma-delimited format

4. **Export Serial Records:**

   ```progress
   EXPORT DELIMITER ',' Serial
   ```

   - Each Serial record exported as one CSV row
   - All 50+ fields included
   - Comma-separated values with proper escaping

5. **Tracking Variables (Per ProductCode):**
   - As report iterates, accumulates totals PER PRODUCT:
     - `intTotCnt`: Total count (Add=+1, Delete=-1)
     - `decLabWgt`: Total label weight
     - `decNetWgt`: Total net weight
     - `decAvgWgt`: Average weight = NetWgt / Count
     - `decTotalGive`: Total giveaway = NetWgt - LabelWgt
     - `decTotalCaseGive`: Average giveaway = TotalGive / Count

6. **CSV Close & Summary:**
   - Close CSV output file
   - Display confirmation message with record count

### 4.2 Procedure: ipReport (Formatted Summary)

This procedure generates the human-readable summary display alongside CSV export.

**Output Structure:**

```
Scale: [ScaleNum]
Shift: [ShiftNum]
--------------------
[Product Code Summary Table]

Scale: [Next ScaleNum]
Shift: [ShiftNum]
--------------------
[Product Code Summary Table]

...

TOTAL: [Grand Total Row]
```

**Per-Product Summary Line:**

```
[ProductCode]  [Count]  [LabelWgt]  [NetWgt]  [AvgWgt]  [Giveaway]  [CaseGiveaway]
12301          145      1,450.32    1,645.78  11.35    195.46      1.35
```

**Column Layout:**

- ProductCode: 5 digits
- Count: Total boxes added minus deleted
- LabelWgt: Sum of all label weights
- NetWgt: Sum of all net weights
- AvgWgt: NetWgt / Count
- Giveaway (Total): NetWgt - LabelWgt
- Giveaway (Per Case): Total Giveaway / Count

**Grouping & Breaks:**

- Primary break: By Scale (horizontal separator added)
- Secondary break: By ProductCode (subtotals calculated)
- Final break: Overall totals across all scales/products

**Display Format:**

- Display added to Selection-List widget (SELECT-1)
- Each line selectable (though non-interactive)
- Allows user to scroll through results
- Provides page navigation buttons if report > 20 lines

---

## 5. Report Output Format

### 5.1 CSV File Structure

**File Location:** `temp/serials-[DateRange]-shift-[Shift].csv`

**Header Row (All Field Names):**

```
SerialNum,AddOrDel,PrintDate,PrintTime,PackDate,KillDate,Shift,Operator,Sent,ScaleNumber,SellByOffset,VarOffset,WgtUnits,LabelWgt,NetWgt,BoxTare,ExtraTare,ProductCode,Lot,GoalID,Reprinted,DateCode,PrintFlag,QCFlag,[... ~30 more fields ...]
```

**Data Row Example:**

```
202401150001,A,01/15/2024,43200,01/15/2024,01/15/2024,1,jsmith,no,1,10,0,LB,7.500,8.250,0.750,0.000,12301,LOT001,5001,no,2024-01-15,yes,no,[...]
202401150002,A,01/15/2024,43201,01/15/2024,01/15/2024,1,jsmith,no,1,10,0,LB,7.500,8.260,0.750,0.000,12301,LOT001,5001,no,2024-01-15,yes,no,[...]
202401150003,D,01/15/2024,43202,01/15/2024,01/15/2024,1,jsmith,no,1,10,0,LB,7.500,8.240,0.750,0.000,12301,LOT001,5001,no,2024-01-15,yes,no,[...]
[... thousands more rows ...]
```

**Field Count:** 50+ fields exported (all Serial table fields)

**Row Count:** One row per Serial record matching filter criteria

**Total Size:** ~500 bytes per record, so 100K records = ~50 MB (typical export: 10-500 MB)

### 5.2 Display Summary Format

**Interactive Selection List Display:**

```
                     Serials Export Summary
Scale: 1
Shift: 1
--------------------
12301        145       1,450.32       1,645.78        11.35      195.46       1.35
12302        98        980.75         1,114.55        11.37      133.80       1.37
12305        52        520.00         589.75          11.34      69.75        1.34
Total        295       2,951.07       3,350.08        11.35      398.01       1.35

Scale: 2
Shift: 1
--------------------
12301        127       1,270.00       1,441.50        11.35      171.50       1.35
12302        84        840.50         953.10          11.35      112.60       1.34
Total        211       2,110.50       2,394.60        11.35      284.10       1.35

GRAND TOTAL: 506  4,961.57       5,744.68        11.35      682.11       1.35
```

**Format Details:**

- ProductCode: Right-justified 5 digits
- Count: Right-justified 9 digits (with leading spaces)
- Weights: Right-justified 12 digits with sign, 2 decimal places
- Average/Variance fields: 10 digits with 2 decimals

---

## 6. Business Rules

### Rule 6.1: Date Range Selection

**Rule:** Users can specify any date range from 1 day to several months

**Details:**

- Start Date must be ≤ End Date
- No validation enforced (users can select invalid ranges)
- Empty range (no matching serials) returns empty CSV file
- System maintains reasonable defaults (last 80 days)

**Implication:** Users responsible for logical date selection

---

### Rule 6.2: Date Type Selection (Pack vs. Print)

**Rule:** Report filters by EITHER PackDate OR PrintDate, not both

**Details:**

- PackDate: When case was physically packed/produced
- PrintDate: When the label was printed (typically same day but may differ)
- Selection is radio-button (mutually exclusive)
- PrintDate selection useful for work-day analysis with FM (False Midnight) logic

**Business Logic:**

```progress
IF rsPackDate THEN
    WHERE Serial.PackDate >= StartDate AND Serial.PackDate <= EndDate
ELSE
    WHERE Serial.PrintDate >= StartDate AND Serial.PrintDate <= EndDate
```

---

### Rule 6.3: Shift Filtering

**Rule:** Can filter by specific shift OR aggregate all shifts

**Details:**

- Shift values: typically 1, 2, 3 (configurable 1-99)
- "All Shifts" option aggregates across all shifts
- "This Shift" option filters to single shift value
- Filter applies: `Serial.Shift = SelectedShift`

---

### Rule 6.4: Reprinted Record Exclusion

**Rule:** Serial records marked as "Reprinted" are EXCLUDED from the report

**Details:**

- When a label is reprinted, original record has Reprinted=YES
- New reprinted record is created with Reprinted=NO
- Report uses: `AND NOT Serial.Reprinted`
- Effect: Only shows valid/current serial records, not overridden ones

**Implication:** If a label was reprinted, export only shows the latest version

---

### Rule 6.5: Add/Delete Tracking

**Rule:** Serial.AddOrDel field tracks whether case was added or cancelled

**Details:**

- AddOrDel='A': Case was added/produced normally
- AddOrDel='D': Case was deleted/cancelled/rejected
- Both types included in CSV export
- Summary rows calculate net count: `+1 for 'A', -1 for 'D'`

**Example:**

```
Count = 100 (Added) - 5 (Deleted) = 95 net cases
```

---

### Rule 6.6: Weight Aggregation

**Rule:** Weights are summed by ProductCode for summary display

**Details:**

```
Per-Product Summary:
  Total Label Weight = SUM(Serial.LabelWgt) for all serials of product X
  Total Net Weight = SUM(Serial.NetWgt) for all serials of product X
  Average Weight = Total Net Weight / Case Count
  Giveaway = Net Weight - Label Weight
  Case Giveaway = Giveaway / Case Count
```

**Tracking Variables:**

- `intTotCnt`: Net case count (with add/delete applied)
- `decLabWgt`: Total label weight
- `decNetWgt`: Total net weight
- `decAvgWgt`: Calculated average weight
- `decTotalGive`: Total giveaway weight
- `decTotalCaseGive`: Average giveaway per case

---

### Rule 6.7: Scale Hierarchy in Display

**Rule:** Summary display groups by Scale, then by ProductCode

**Details:**

- Primary grouping: By `Serial.Scale` (equipment identifier)
- Secondary grouping: By `Serial.ProductCode` (product)
- Subtotals for each ProductCode within Scale
- Subtotals for each Scale
- Grand total across all scales

**Display Structure:**

```
Scale: 1
  Product 12301: [subtotal]
  Product 12302: [subtotal]
  Scale 1 Total: [subtotal]
Scale: 2
  Product 12301: [subtotal]
  Product 12302: [subtotal]
  Scale 2 Total: [subtotal]
GRAND TOTAL: [total all scales]
```

---

### Rule 6.8: CSV Export Completeness

**Rule:** CSV file contains ALL Serial table fields, dynamically retrieved

**Details:**

- Column headers built from database metadata (\_file, \_field tables)
- Ensures all fields present regardless of future schema changes
- Fields exported in database definition order (by ORDER field)
- Allows recipient to properly interpret all data

**Implication:** CSV file structure changes if Serial table schema modified

---

### Rule 6.9: CSV Field Ordering

**Rule:** Field order in CSV matches database schema definition order

**Details:**

- Source: \_field table ORDER field for Serial table
- Typical order: SerialNum, AddOrDel, PrintDate, PrintTime, PackDate, KillDate, Shift, ...
- Column header row reflects this order
- Allows automated parsing by field position

---

### Rule 6.10: No Data Filtering Beyond Parameters

**Rule:** Only three filters applied: DateRange, DateType, Shift, Reprinted flag

**Details:**

- NO filtering by: Product, Operator, Scale (all included)
- NO filtering by: Weight ranges, Add/Delete status (all included)
- This gives maximum data export for flexibility

**Implication:** Recipient must apply secondary filtering if needed

---

### Rule 6.11: File Naming Convention

**Rule:** CSV filename encodes selection parameters for easy identification

**Format:** `serials-[MMDDYY-to-MMDDYY]-shift-[1|2|3|all].csv`

**Examples:**

- `serials-01-15-24-to-01-31-24-shift-1.csv` (Jan 15-31, Shift 1)
- `serials-01-15-24-to-01-31-24-shift-all.csv` (Jan 15-31, All Shifts)

---

### Rule 6.12: Temporary File Storage

**Rule:** CSV files stored in `temp/` directory (Windows: `temp\`)

**Details:**

- Not permanent archive (subject to cleanup)
- User should copy to permanent location if needed
- May be overwritten if same date range/shift selected again

---

### Rule 6.13: No Aggregation or Averaging in CSV

**Rule:** CSV export contains raw detail records only (no aggregation)

**Details:**

- Each Serial record = one CSV row
- No summary rows
- No aggregation fields (those are in display only)
- CSV preserves original data integrity

**Implication:** Recipient must perform aggregation in Excel/BI tool if needed

---

### Rule 6.14: Sent Flag Preservation

**Rule:** Serial.Sent flag is preserved in CSV; not filtered

**Details:**

- Sent=YES: Record has been exported to external system
- Sent=NO: Record not yet sent
- Both included in CSV export
- Allows tracking of data synchronization

---

### Rule 6.15: Record Deduplication

**Rule:** Only non-reprinted records included (see Rule 6.4)

**Details:**

- If Serial reprinted: original marked Reprinted=YES, new created Reprinted=NO
- Filter: `NOT Serial.Reprinted` excludes old versions
- Prevents duplicate/stale data in export

---

## 7. Relationships to Other Concepts

### Serial Table Relationships

**Serial ← Goal (many-to-one)**

- Serial.GoalID references Goal record
- Serials produced as part of production goal
- Allows tracing cases to specific production orders
- Goal contains target quantity, scheduling, etc.

**Serial ← Product (many-to-one)**

- Serial.ProductCode references Product
- Serials contain Product weight specifications
- Allows cross-reference to product master data

**Serial ← Operator (many-to-one)**

- Serial.Operator field stores username
- References Operator table (who printed label)
- Allows tracking which staff member processed case

**Serial ← Serial\* (self-referencing)**

- When label reprinted, new Serial created with old row marked Reprinted=YES
- Allows audit trail of label reprints
- Original vs. final version tracking

### Related Tables & Views

**ItemSerial Table:**

- Relationship: Each Serial may contain multiple ItemSerials (items packed into case)
- Link: Serial.SerialNum = ItemSerial.SerialNum (partial)
- Purpose: Detailed item-level breakdown within each case
- Use Case: Quality verification of items within a case

**Modify Table:**

- Relationship: Production-time weight adjustments
- Link: Modify.SerialNum references Serial
- Purpose: Tare overrides, weight corrections made during production
- Use Case: When standard tare doesn't match actual box weight

**BatchOrder Table:**

- Relationship: Serials grouped into batch/order
- Link: Serial records for same production run/batch
- Purpose: Order grouping, customer identification
- Use Case: Fulfilling specific customer orders

**Totals Table:**

- Relationship: Aggregated daily summaries from Serials
- Link: Totals derived by aggregating Serials by ProductCode/Shift/Date
- Purpose: Daily production reporting (while Serials = detail)
- Use Case: Quick daily summary vs. detailed case analysis

---

## 8. Common Queries & Data Interpretation

### Query 1: Export All Production for a Date Range

**Question:** "I need all detailed case data for January 15-31. Export to analyze in Excel."

**Process:**

1. Open Serials to CSV Report
2. Set Start Date = 01/15/2024
3. Set End Date = 01/31/2024
4. Select "Pack Date"
5. Select "All Shifts"
6. Click "Run Report"
7. CSV file created: `temp/serials-01-15-24-to-01-31-24-shift-all.csv`
8. Copy file to permanent location for analysis

**Result:** CSV with 10,000+ rows (all cases for 17 days, all shifts)

---

### Query 2: Shift-by-Shift Comparison

**Question:** "Which shift produced the most cases yesterday? Show details for each shift."

**Process:**

1. **Shift 1 Export:**
   - Start Date = Yesterday
   - End Date = Yesterday
   - Select "Pack Date"
   - Select "This Shift" = 1
   - Run Report → Display shows Shift 1 summary
   - CSV file: `temp/serials-[date]-shift-1.csv`
   - Note: Total case count and total weights

2. **Shift 2 Export:**
   - Same dates
   - Select "This Shift" = 2
   - Run Report → Display shows Shift 2 summary
   - CSV file: `temp/serials-[date]-shift-2.csv`

3. **Shift 3 Export:**
   - Same dates
   - Select "This Shift" = 3
   - Run Report → Display shows Shift 3 summary
   - CSV file: `temp/serials-[date]-shift-3.csv`

**Comparison:**

```
Shift 1: 250 cases  |  Shift 2: 280 cases  |  Shift 3: 210 cases
```

**Winner:** Shift 2 with 280 cases

---

### Query 3: Quality Audit - Weight Compliance

**Question:** "Are all cases of product 12301 within weight specification (7.5-8.5 oz)?"

**Process:**

1. Export all recent cases for product 12301 using Serials to CSV
2. Open CSV in Excel
3. Filter for ProductCode = 12301
4. Create formula to check: NetWgt >= 7.50 AND NetWgt <= 8.50
5. Flag any out-of-spec cases

**Potential Issues Identified:**

- Serial 202401150047: NetWgt = 6.85 oz (UNDER SPEC)
- Serial 202401150052: NetWgt = 8.65 oz (OVER SPEC)

**Action:** Investigate these specific cases for quality issues

---

### Query 4: Deleted Cases Analysis

**Question:** "How many cases were cancelled/rejected today? Why?"

**Process:**

1. Export today's data
2. Filter CSV for AddOrDel = 'D'
3. Count deleted records
4. Review comment/reason fields (if populated)

**Example Result:**

```
Total cases produced: 250
Deleted cases: 8 (3.2%)
Reason codes: Weight out of spec (5), Label defect (2), Equipment error (1)
```

---

### Query 5: Exported Data Synchronization

**Question:** "Which cases have been sent to the external system vs. not yet sent?"

**Process:**

1. Export full data range
2. Open CSV in Excel
3. Filter Sent column: YES vs. NO
4. Count each group

**Result:**

```
Sent = YES: 8,500 cases (already synchronized)
Sent = NO: 150 cases (pending synchronization)
```

**Action:** Run sync process to send remaining 150 cases

---

## 9. Integration Points

### 1. Integration with Excel/Spreadsheets

**CSV → Excel Workflow:**

1. Generate CSV from Serials to CSV Report
2. Open CSV file in Excel
3. Auto-format as table
4. Create pivot tables for analysis
5. Add formulas for weight compliance checks
6. Generate charts for production trends

**Common Analysis:**

- Average weight by shift
- Defect rates by operator
- Production efficiency by scale
- Weight variance analysis

---

### 2. Integration with Database Systems

**CSV → Database ETL:**

1. Export CSV from Serials to CSV Report
2. Load into staging table in DataWarehouse
3. Validate data quality
4. Transform/clean data
5. Load into production analytics database
6. Query for BI reports/dashboards

**Database Schema:**

```sql
CREATE TABLE SerialArchive (
  SerialNum VARCHAR(12),
  ProductCode INT,
  NetWgt DECIMAL(7,3),
  LabelWgt DECIMAL(7,3),
  PackDate DATE,
  Shift INT,
  ...
)
```

---

### 3. Integration with BI Tools (Power BI, Tableau, etc.)

**CSV → BI Tool Workflow:**

1. Export CSV to shared location
2. Connect BI tool to CSV (scheduled refresh)
3. Build dashboards showing:
   - Production by product/shift/date
   - Weight trends and specification compliance
   - Quality metrics (defect rates)
   - Operator performance
4. Share interactive reports with stakeholders

**Sample Dashboard:**

```
┌─────────────────────────────────┐
│ Production Dashboard (Real-time)│
├─────────────────────────────────┤
│ Total Cases: 5,250              │
│ Avg Weight: 11.35 oz            │
│ Within Spec: 99.8%              │
│ Shift 1: 1,800  Shift 2: 1,950  │
│ Shift 3: 1,500                  │
└─────────────────────────────────┘
```

---

### 4. Integration with Quality Management Systems

**CSV → QMS Workflow:**

1. Export production cases as CSV
2. Import into Quality Management System
3. QMS performs:
   - Statistical analysis (SPC)
   - Trend analysis
   - Out-of-spec detection
   - Compliance reporting
4. Generate quality reports for certification

**QMS Uses:**

- ISO/HACCP compliance documentation
- Metric tracking (weight variance, defects)
- Audit trail for regulatory proof
- Certification support

---

### 5. Integration with ERP Systems

**CSV → ERP Sync:**

1. Export Serial data as CSV
2. Parse and format for ERP
3. Load into ERP inventory/production modules
4. Link to:
   - Sales orders (customer fulfillment)
   - Lot tracking (traceability)
   - Cost accounting (weight-based pricing)
5. Generate ERP reports

**ERP Benefits:**

- Inventory accuracy
- Customer lot assignment
- Weight-based billing
- Production costing

---

## 10. Performance Considerations

### Dataset Size Impact

**Typical Scenarios:**

| Scenario                 | Record Count   | File Size        | Export Time   |
| ------------------------ | -------------- | ---------------- | ------------- |
| Single day, 1 shift      | 200-500        | 100-250 KB       | <1 second     |
| One week, all shifts     | 1,500-3,500    | 750 KB - 1.75 MB | 2-5 seconds   |
| One month, all shifts    | 6,000-15,000   | 3-7.5 MB         | 10-30 seconds |
| Three months, all shifts | 18,000-45,000  | 9-22.5 MB        | 30-90 seconds |
| One year, all shifts     | 72,000-180,000 | 36-90 MB         | 2-5 minutes   |

**Factors Affecting Performance:**

- Date range (larger range = more records)
- All shifts (includes all) vs. single shift (fewer records)
- Database disk I/O speed
- System load
- Available memory

### Memory & Storage

**RAM Usage:**

- Typical export uses <100 MB RAM
- Large exports (>100K records) may use 200-500 MB
- System has sufficient memory for largest exports

**Disk Space:**

- `temp/` directory should have 100+ MB free
- CSV files may accumulate if not cleaned up
- Consider archiving old CSV files

**Network Considerations:**

- If temp location is network drive: slower (may add 50% to time)
- Local `temp/` directory: fastest
- Recommend using local temp for large exports

### Optimization Opportunities

1. **Date Range Limiting**: Export shorter ranges when possible
2. **Shift Filtering**: Export single shift instead of all shifts if possible
3. **Scheduled Exports**: Schedule large exports during off-hours
4. **Archive Strategy**: Move old CSV files to archive after use
5. **Query Optimization**: Database indexes on PackDate/PrintDate/Shift improve performance

### Scalability Limits

**Current Implementation Can Handle:**

- ✅ Single exports up to 1 year of data (100K+ records)
- ✅ Multiple concurrent exports (OS dependent)
- ✅ File sizes up to 100+ MB (disk permitting)

**Potential Limitations:**

- ❌ Very large exports (>500K records) may be slow
- ❌ Network-based temp directory slows performance
- ❌ Very old data (2+ years) may have fragmentation
- ❌ Running multiple large exports simultaneously

**Solutions for Large Exports:**

1. Break into multiple smaller date ranges
2. Filter by shift
3. Archive old data to separate table
4. Use database-native export tools for very large datasets

---

## 11. Troubleshooting

### Issue: CSV File Not Created

**Possible Causes:**

1. Invalid date range (start date > end date)
2. No matching records (no serials in date range)
3. File system error (temp directory not writable)
4. Disk full or permission denied

**Debugging Steps:**

1. Verify Start Date ≤ End Date
2. Check if there are active serials in the date range (query Serial table)
3. Verify `temp/` directory exists and is writable
4. Check free disk space (need at least 100 MB)
5. Check user permissions on temp directory
6. Try selecting shorter date range (e.g., 1 day vs. 3 months)

---

### Issue: CSV File Is Empty or Has Only Headers

**Possible Causes:**

1. No Serial records match the filter criteria
2. All matching serials marked as Reprinted=YES
3. Database connection lost during export

**Debugging Steps:**

1. Verify serials exist for selected date range
   ```progress
   FOR EACH Serial NO-LOCK
       WHERE Serial.PackDate = [selected date]
   ```
2. Check if records are marked Reprinted
   ```progress
   FOR EACH Serial NO-LOCK WHERE Reprinted = YES
   ```
3. Try shorter date range to confirm connectivity
4. Check database logs for connection errors

---

### Issue: Summary Display Not Showing (Blank Selection List)

**Possible Causes:**

1. CSV export successful but display not populated
2. Selection list widget not properly initialized
3. Summary calculation encountered error

**Debugging Steps:**

1. Check if CSV file was created (check temp/ directory)
2. Verify CSV file is not empty (open with text editor)
3. Review error messages in progress logs
4. Try running Report again
5. Close and reopen Report window

---

### Issue: CSV File Has Incorrect Data (Wrong Date Range)

**Possible Causes:**

1. Date parameter not properly captured
2. Cached data from previous run
3. User error in date selection

**Debugging Steps:**

1. Verify date fields show correct values in Report window
2. Check CSV filename encodes correct dates
3. Sample CSV data - verify dates match expectation
4. Re-run report with explicit dates

---

### Issue: Weight Values Show as Zero or Placeholder

**Possible Causes:**

1. Serial records have null/zero weight values
2. Weight fields not populated during label printing
3. CSV export formatting issue

**Debugging Steps:**

1. Check raw Serial table records for weight values
2. Verify label printing process populates LabelWgt and NetWgt
3. Examine CSV file in text editor (check for placeholders)
4. Sample a few serial numbers and query directly

---

### Issue: Missing Serials in Export (Expected Cases Not Appearing)

**Possible Causes:**

1. Cases marked as Reprinted (excluded by filter)
2. Cases outside selected date range (wrong date type selected)
3. Cases on different shift than selected
4. Database query filtering too aggressively

**Debugging Steps:**

1. Check if cases marked Reprinted=YES
2. Verify using correct date type (PackDate vs. PrintDate)
3. Check shift selection (All Shifts vs. specific)
4. Query Serial table directly with same WHERE criteria
5. Check if Serial records deleted after production

---

### Issue: Performance Slow (Large Export Takes Too Long)

**Possible Causes:**

1. Large date range (months of data)
2. Database slow or system under load
3. Network filesystem (if temp is network location)
4. Large number of serials in period

**Solutions:**

1. Break into smaller date ranges (week at a time vs. month)
2. Select specific shift instead of all shifts
3. Use local temp directory instead of network
4. Run during off-hours (less system load)
5. Archive very old data to separate table
6. Consider database optimization (index maintenance)

---

## 12. Data Dictionary - Serial Table Fields

### Core Identity Fields

| Field         | Type    | Size | Description                            | Example      |
| ------------- | ------- | ---- | -------------------------------------- | ------------ |
| **SerialNum** | CHAR    | 12   | Unique 12-digit serial number          | 202401150001 |
| **AddOrDel**  | CHAR    | 1    | 'A'=Added, 'D'=Deleted/Cancelled       | A or D       |
| **Reprinted** | LOGICAL | 1    | Label reprinted (excluded from report) | yes/no       |

### Date & Time Fields

| Field         | Type | Size | Description                   | Example          |
| ------------- | ---- | ---- | ----------------------------- | ---------------- |
| **PrintDate** | DATE | 4    | Date label was printed        | 01/15/2024       |
| **PrintTime** | INT  | 4    | Time label printed (seconds)  | 43200 (12:00:00) |
| **PackDate**  | DATE | 4    | Date case packed/produced     | 01/15/2024       |
| **KillDate**  | DATE | 4    | Date product killed/processed | 01/15/2024       |
| **DateCode**  | CHAR | 10   | Formatted date code           | 2024-01-15       |

### Production Fields

| Field           | Type | Size | Description               | Example    |
| --------------- | ---- | ---- | ------------------------- | ---------- |
| **ProductCode** | INT  | 5    | Product ID (00001-99999)  | 12301      |
| **Shift**       | INT  | 2    | Shift number              | 1, 2, or 3 |
| **ScaleNumber** | INT  | 2    | Scale/equipment ID        | 1          |
| **Operator**    | CHAR | 10   | Operator/user ID          | JSMITH     |
| **GoalID**      | INT  | 4    | Production goal reference | 5001       |
| **Lot**         | CHAR | 10   | Lot number                | LOT001     |

### Weight Fields

| Field         | Type | Size | Description                | Example |
| ------------- | ---- | ---- | -------------------------- | ------- |
| **LabelWgt**  | DEC  | 7,3  | Label/package tare weight  | 7.500   |
| **NetWgt**    | DEC  | 7,3  | Net product weight         | 8.250   |
| **BoxTare**   | DEC  | 7,3  | Box/container tare         | 0.750   |
| **ExtraTare** | DEC  | 7,3  | Additional tare (moisture) | 0.000   |
| **WgtUnits**  | CHAR | 2    | Weight units (LB or KG)    | LB      |

### Date Code Fields

| Field            | Type | Size | Description                  | Example |
| ---------------- | ---- | ---- | ---------------------------- | ------- |
| **SellByOffset** | INT  | 2    | Days offset for sell-by date | 10      |
| **VarOffset**    | INT  | 2    | Variable offset from today   | 0       |

### System Fields

| Field         | Type    | Size | Description                 | Example |
| ------------- | ------- | ---- | --------------------------- | ------- |
| **Sent**      | LOGICAL | 1    | Exported to external system | yes/no  |
| **PrintFlag** | LOGICAL | 1    | Print status                | yes/no  |
| **QCFlag**    | LOGICAL | 1    | Quality control flag        | yes/no  |

_Note: Additional ~30 fields available in full CSV export (see complete field list in Serial table definition)_

---

## 13. Business Rules Summary Table

| Rule # | Rule Name              | Selection              | Impact                             |
| ------ | ---------------------- | ---------------------- | ---------------------------------- |
| 6.1    | Date Range Selection   | User configurable      | Determines serial records included |
| 6.2    | Date Type Selection    | PackDate vs PrintDate  | Changes filter logic               |
| 6.3    | Shift Filtering        | All or specific shift  | Focuses on shift-level analysis    |
| 6.4    | Reprinted Exclusion    | Automatic filter       | Removes duplicate/old records      |
| 6.5    | Add/Delete Tracking    | Preserved in data      | Net count affects totals           |
| 6.6    | Weight Aggregation     | Per-product summaries  | Calculated in display only         |
| 6.7    | Scale Hierarchy        | Display grouping       | Organizes summary view             |
| 6.8    | CSV Completeness       | All fields exported    | Future-proof schema changes        |
| 6.9    | Field Ordering         | Schema-based order     | Matches database definition        |
| 6.10   | No Extra Filtering     | Minimal filters only   | Maximizes export flexibility       |
| 6.11   | File Naming Convention | [Dates]-shift-[Shift]  | Self-documenting filenames         |
| 6.12   | Temporary Storage      | temp/ directory        | Subject to cleanup                 |
| 6.13   | CSV Detail Only        | No aggregation in file | Recipient performs analysis        |
| 6.14   | Sent Flag Preservation | Included in export     | Tracking of synchronization        |
| 6.15   | Record Deduplication   | Reprinted=NO only      | Latest version only                |

---

---

## Appendix A: User Quick Start

### Running the Serials to CSV Report

1. **Navigate to Reports:** Main Menu → Reports → Serials to CSV
2. **Set Date Range:**
   - Start Date: Enter beginning date (default: 80 days ago)
   - End Date: Enter ending date (default: today)
   - Use Up/Down buttons to adjust by single days
3. **Select Date Type:**
   - ○ Pack Date (when produced)
   - ○ Print Date (when labeled)
4. **Select Shift:**
   - ○ All Shifts (all production)
   - ○ This Shift (select 1, 2, or 3)
5. **Click "Run Report"**
   - CSV file created in `temp/serials-...csv`
   - Summary displayed on screen
   - Note the total record count
6. **Optional: Print Report**
   - Click "Print Report" to send summary to printer
7. **Copy CSV File**
   - Open file manager and navigate to `temp/`
   - Copy `serials-*.csv` to permanent location
   - Open in Excel, import to database, or send to BI tool

### Key Information to Note

- **CSV Filename:** Encodes date range and shift selection
- **Record Count:** Number of real serial records (shown in confirmation)
- **Grand Total Count:** Sum of all cases (added minus deleted)
- **Weights:** All in specified units (LB or KG)
- **Reprinted Cases:** Excluded automatically (only current versions)

---

**End of Document**
