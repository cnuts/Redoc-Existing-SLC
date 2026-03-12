
## 1. Overview

### What Is a Department?

A **Department** is an **organizational and operational unit** within a manufacturing facility that represents a physical production area, line, or functional zone. It's the logical grouping of:

- **Physical equipment** (scales, conveyor systems, labeling machines)
- **Operators** (assigned to work shifts within the department)
- **Production capacity** (how many cases per hour it can process)
- **Quality checkpoints** (QC procedures specific to that department)
- **Output devices** (printers for label production)

Think of it like dividing a meat processing facility into:
- **Department GB01:** Ground Beef Processing (grinding, mixing)
- **Department PKG01:** Packaging (filling containers, sealing)
- **Department QUALITY:** Quality Control (final weight verification)
- **Department SHIP01:** Shipping (palletization, loading)

### Why Departments Exist

Manufacturing facilities need to:

- **Track Production Location** - Know which department produced each case
- **Manage Equipment** - Assign scales to specific departments
- **Route Work** - Move products through departments in sequence
- **Measure Performance** - Compare efficiency across departments
- **Identify Bottlenecks** - See which department limits overall throughput
- **Organize Labor** - Assign operators to departments per shift
- **Allocate Costs** - Calculate labor/equipment costs per department
- **Plan Capacity** - Know how many cases each department can handle
- **Ensure Quality** - Apply QC procedures specific to each department step
- **Compliance** - Track production by location for regulatory requirements (FDA, USDA)

---

## 2. Department Structure

### Implicit vs. Explicit Design

**In your SLC application, Department is IMPLICIT** (not stored as a separate master table). Instead:

- **Departments are defined through Scale assignments**
- Each **Scale belongs to ONE Department**
- When a **Serial is created at a Scale**, it inherits the department context
- **System derives department from: Serial.ScaleNumber → Scale.Department**

### Conceptual Department Table

While a dedicated `Department` table may not exist explicitly, the concept is managed through:

```sql
/* Conceptual Department structure (referenced through Scale) */
Department = {
    DepartmentID        CHAR(10) NOT NULL PRIMARY KEY,  -- e.g., "GB01", "PKG01"
    DepartmentName      CHAR(50) NOT NULL,              -- e.g., "Ground Beef Line"
    FacilityID          CHAR(10),                       -- Which facility
    Location            CHAR(50),                       -- Physical location in plant
    ScaleRange          (StartScale INT, EndScale INT), -- Scales in this dept
    LabelPrinter        CHAR(20),                       -- Primary output device
    CapacityCasesPerHr  INT,                            -- Max throughput
    MinOperators        INT,                            -- Minimum crew size
    MaxOperators        INT,                            -- Maximum crew size
    Active              LOGICAL DEFAULT YES             -- Operational status
}
```

### Scale - Department Association

**Scale Table** is the explicit table that links to Department:

```sql
CREATE TABLE Scale (
    ScaleNumber         INT NOT NULL PRIMARY KEY,       -- Physical scale ID (1, 2, 3...)
    ScaleDescription    CHAR(50),                       -- e.g., "GB-Scale-A"
    Department          CHAR(10),                       -- FK to Department
    Location            CHAR(50),                       -- Physical location
    LabelPrinter        CHAR(20),                       -- Printer name
    ScaleType           CHAR(10),                       -- Type of scale
    CalibrationDate     DATE,                           -- Last calibration
    Status              CHAR(10),                       -- ACTIVE, DOWN, MAINTENANCE
    Facility            CHAR(10)                        -- Which facility
)
```

**Example Scales by Department:**

```
Department: GB01 (Ground Beef Line)
  ├─ Scale 1: "GB-Scale-A (Intake weighing)" → Printer: LBL-PRINTER-01
  ├─ Scale 2: "GB-Scale-B (Secondary check)" → Printer: LBL-PRINTER-01
  └─ Scale 3: "GB-Scale-C (Final pack)" → Printer: LBL-PRINTER-02

Department: PKG01 (Packaging)
  ├─ Scale 4: "PKG-Scale-A (Case assembly)" → Printer: LBL-PRINTER-03
  └─ Scale 5: "PKG-Scale-B (Final weight)" → Printer: LBL-PRINTER-03

Department: QUALITY (Quality Control)
  └─ Scale 6: "QC-Scale (Verification)" → Printer: LBL-PRINTER-04
```

---

## 3. Department Categories

### By Production Type

**Processing Departments:**

```
Ground Beef Line (GB01)
  ├─ Input: Live animals or bulk meat
  ├─ Process: Grinding, mixing, portioning
  ├─ Output: Ground beef in cases
  ├─ Scales: 1-3
  ├─ Capacity: 500 cases/hour
  ├─ Printer: LBL-PRINTER-01
  └─ Quality Gate: Temperature, visual inspection, metal detection
```

**Packaging Departments:**

```
Case Assembly (PKG01)
  ├─ Input: Ground beef from GB01
  ├─ Process: Placing into containers, weighing, sealing
  ├─ Output: Packaged cases
  ├─ Scales: 4-5
  ├─ Capacity: 400 cases/hour
  ├─ Printer: LBL-PRINTER-03
  └─ Quality Gate: Weight verification, seal integrity
```

**Quality Control Departments:**

```
QC/Final Check (QUALITY)
  ├─ Input: Packaged cases from PKG01
  ├─ Process: Weight verification, defect inspection
  ├─ Output: Approved cases for shipping
  ├─ Scales: 6
  ├─ Capacity: 300 cases/hour
  ├─ Printer: LBL-PRINTER-04
  └─ Quality Gate: Final weight confirmation, hold/release authority
```

**Support Departments:**

```
Shipping/Staging (SHIP01)
  ├─ Input: Approved cases from QA
  ├─ Process: Consolidation, pallet building, palletization
  ├─ Output: Shipment-ready pallets
  ├─ Scales: None (no individual case weighing)
  ├─ Capacity: 2,000 cases/day
  ├─ Printer: None (barcode scanning only)
  └─ Quality Gate: Pallet weight verification
```

### By Facility Structure

**Single Facility (Simple Layout):**

```
Plant: MIAMI
  ├─ Department GB01 (Ground Beef)
  ├─ Department PKG01 (Packaging)
  └─ Department QUALITY (QC)
```

**Multi-Line Facility (Complex Layout):**

```
Plant: CHICAGO
  ├─ Building A - Ground Beef Production
  │   ├─ Department GB01 (Line 1, Scales 1-3)
  │   ├─ Department GB02 (Line 2, Scales 4-6)
  │   └─ Department GB03 (Line 3, Scales 7-9)
  │
  ├─ Building B - Packaging
  │   ├─ Department PKG01 (Packing Line 1, Scales 10-11)
  │   ├─ Department PKG02 (Packing Line 2, Scales 12-13)
  │   └─ Department PKG03 (Packing Line 3, Scales 14-15)
  │
  └─ Building C - QC & Shipping
      ├─ Department QUALITY (QC Station, Scale 16)
      └─ Department SHIP01 (Shipping, no scales)
```

---

## 4. Department and Scale Relationship

### Scale Assignment to Department

Each **Scale** is assigned to exactly ONE **Department**:

```
Scale → (belongs to) → Department (1:N relationship)

Department "GB01" contains:
  ├─ Scale 1 (Intake weighing)
  ├─ Scale 2 (Secondary check)
  └─ Scale 3 (Final pack weight)

Department "PKG01" contains:
  ├─ Scale 4 (Case assembly weighing)
  └─ Scale 5 (Final shipping weight)
```

### Scale Number to Department Mapping

System maintains implicit mapping:

```
Scale 1 → Department GB01
Scale 2 → Department GB01
Scale 3 → Department GB01
Scale 4 → Department PKG01
Scale 5 → Department PKG01
Scale 6 → Department QUALITY
```

**Lookup Logic:**

```progress
/* Find department for a given scale */
FIND Scale WHERE Scale.ScaleNumber = input-scale-num NO-ERROR.
IF AVAILABLE Scale THEN
    v-DepartmentID = Scale.Department.
```

---

## 5. Serial Records and Department

### Serial Creation and Department Context

When a Serial is created, it implicitly captures department context:

```
Operator scans product at Scale 2
  → Scale 2 belongs to Department GB01
  → System determines: Department = "GB01"

Serial record created:
  ├─ SerialNum = "20260311-001"
  ├─ ScaleNumber = 2          ← Identifies department via scale
  ├─ Department = "GB01"      ← Derived from ScaleNumber
  ├─ ProductCode = 1001
  ├─ Operator = "jsmith"
  └─ PrintDate = 03/11/2026
```

### Query Serial by Department

**Find all units produced by department today:**

```sql
SELECT SerialNum, ProductCode, NetWgt, Operator, PrintTime
FROM Serial
WHERE ScaleNumber IN (
    SELECT ScaleNumber FROM Scale WHERE Department = 'GB01'
)
  AND PrintDate = TODAY
ORDER BY PrintTime;

Result:
  SerialNum    ProductCode  NetWgt  Operator  PrintTime
  ABC-001      1001         8.25    jsmith    06:15:30
  ABC-002      1001         8.30    jsmith    06:16:45
  ABC-003      1001         8.20    jsmith    06:18:10
  ...
```

**Department Production Summary:**

```sql
SELECT 
    CASE WHEN ScaleNumber IN (1, 2, 3) THEN 'GB01'
         WHEN ScaleNumber IN (4, 5) THEN 'PKG01'
         WHEN ScaleNumber = 6 THEN 'QUALITY'
    END as Department,
    COUNT(SerialNum) as TotalCases,
    SUM(NetWgt) as TotalWeight,
    AVG(NetWgt) as AvgWeight,
    MIN(PrintTime) as StartTime,
    MAX(PrintTime) as EndTime
FROM Serial
WHERE PrintDate = TODAY
GROUP BY Department
ORDER BY COUNT(SerialNum) DESC;

Result:
Department  Cases  TotalWeight  AvgWeight  StartTime  EndTime
GB01        450    3,682.50     8.18       06:15:30   14:30:45
PKG01       425    3,462.50     8.15       08:30:15   14:45:20
QUALITY     420    3,425.00     8.15       10:15:00   15:00:30
```

---

## 6. Relationships to Other Concepts

### Department ← Scale (Equipment Assignment)

```
Department (1:N) ──→ Scale

One Department contains MANY Scales
Each Scale belongs to ONE Department

Example:
  Department "GB01"
    → Scale 1 (Intake)
    → Scale 2 (Secondary)
    → Scale 3 (Final)
    → Printer: LBL-PRINTER-01
    → Capacity: 500 cases/hour
```

**Use Case: Scale Performance Analysis**

```
Query: Which scale in department GB01 processed most cases today?

SELECT 
    s.ScaleNumber,
    s.ScaleDescription,
    COUNT(sr.SerialNum) as CasesProcessed,
    AVG(sr.NetWgt) as AvgWeight
FROM Scale s
LEFT JOIN Serial sr ON s.ScaleNumber = sr.ScaleNumber 
  AND sr.PrintDate = TODAY
WHERE s.Department = 'GB01'
GROUP BY s.ScaleNumber, s.ScaleDescription
ORDER BY CasesProcessed DESC;

Result:
ScaleNumber  Description            CasesProcessed  AvgWeight
3            GB-Scale-C (Final)     180            8.15
2            GB-Scale-B (Secondary) 170            8.18
1            GB-Scale-A (Intake)    100            8.20
```

### Department ← Serial (Production Tracking)

```
Department (implicit via Scale) → Serial

Serial records track which scale → implicitly which department

Example:
  Serial ABC-001: ScaleNumber=2 → Department=GB01
  Serial ABC-002: ScaleNumber=4 → Department=PKG01
  Serial ABC-003: ScaleNumber=6 → Department=QUALITY

Enables:
  - Production totals by department
  - Equipment utilization by department
  - Quality issues by department
  - Shift performance by department
```

### Department ← Operator (Shift Assignment)

```
Department --operators-assigned-to--> Shift

Operators are assigned to departments for shifts:

Operator "jsmith"
  - Assigned to Department GB01, Shift 1 (6 AM - 2 PM)
  - Works at scales 1, 2, 3

Operator "mgarcia"
  - Assigned to Department PKG01, Shift 2 (2 PM - 10 PM)
  - Works at scales 4, 5

Daily Production Report by Department & Operator:
┌──────────────────────────────────────────┐
│ Department GB01 - Shift 1 (6 AM - 2 PM)  │
├──────────────────────────────────────────┤
│ Operator: jsmith  - 200 cases           │
│ Operator: rrodriguez - 150 cases        │
│ Total: 350 cases                         │
└──────────────────────────────────────────┘
```

### Department ← Goals (Production Planning)

```
Goals can be created at department level:

Goal 5001
  - Department: GB01
  - ProductCode: 1001
  - TargetQty: 500 cases
  - ScheduledDate: 2026-03-11
  - Shift: 1
  - Status: ACTIVE

When operators produce in GB01 on this shift:
  → Serial records link to Goal 5001
  → Progress tracked: "250 of 500 cases complete (50%)"
  → Achievement measured: GB01 vs. other departments
```

### Department ← PrintDevice (Label Output)

```
Department --has-primary-printer--> PrintDevice

Each Department assigned specific printer(s):

Department GB01
  ├─ Primary Printer: LBL-PRINTER-01 (scales 1, 2)
  └─ Secondary Printer: LBL-PRINTER-02 (scale 3)

Department PKG01
  └─ Printer: LBL-PRINTER-03 (scales 4, 5)

Department QUALITY
  └─ Printer: LBL-PRINTER-04 (scale 6)

Configuration:
  Scale 1 → Department: GB01 → Printer: LBL-PRINTER-01
  Scale 2 → Department: GB01 → Printer: LBL-PRINTER-01
  Scale 3 → Department: GB01 → Printer: LBL-PRINTER-02
  Scale 4 → Department: PKG01 → Printer: LBL-PRINTER-03
  ...

Effect: Labels print to correct physical location automatically
```

### Department ← Quality Control (QC Processes)

```
Department-Specific QC:

Department GB01 (Ground Beef)
  - QC checks: Visual inspection, temperature, color
  - Metal detection: Before packaging

Department PKG01 (Packaging)
  - QC checks: Weight verification, seal integrity
  - Metal detection: After packaging

Department QUALITY
  - Final verification: Weight double-check
  - Defect reconciliation: Hold/release authority

QC records tie to department:
  Serial ABC-001 (ScaleNumber=2, Dept=GB01)
    → QC Check: Visual inspection - PASS
    → Next → Dept PKG01

  Serial ABC-001 (in PKG01 now, ScaleNumber=4)
    → QC Check: Weight verification - PASS
    → Next → Dept QUALITY
```

### Department ← Capacity Planning (Equipment Throughput)

```
Department Capacity Analysis:

Department GB01
  ├─ Capacity: 500 cases/hour
  ├─ Current Production: 350 cases/hour (70%)
  ├─ Remaining Capacity: 150 cases/hour
  └─ Status: Able to accept more work

Department PKG01
  ├─ Capacity: 400 cases/hour
  ├─ Current Production: 380 cases/hour (95%)
  ├─ Remaining Capacity: 20 cases/hour
  └─ Status: BOTTLENECK - at capacity limits

Department QUALITY
  ├─ Capacity: 300 cases/hour
  ├─ Current Production: 250 cases/hour (83%)
  ├─ Remaining Capacity: 50 cases/hour
  └─ Status: Adequate

Implication:
  - PKG01 is bottleneck, limits overall throughput
  - GB01 could increase production if downstream can handle it
  - QUALITY has some buffer capacity
  - Facility maximum = min(GB01, PKG01, QUALITY) = 300-400 CPH
```

### Department ← Cost Accounting (Financial Tracking)

```
Allocate Production Costs by Department:

Daily Labor Costs:
  Department GB01 (Ground Beef): 3 operators × 8 hours × $20/hr = $480
  Department PKG01 (Packaging): 3 operators × 8 hours × $20/hr = $480
  Department QUALITY (QC): 1 operator × 8 hours × $20/hr = $160

Equipment Costs (depreciation):
  Department GB01: Scales 1, 2, 3: $100/day
  Department PKG01: Scales 4, 5: $80/day
  Department QUALITY: Scale 6: $40/day

Total Cost to Produce 350 cases in GB01:
  Labor: $480
  Equipment: $100
  Total: $580
  Cost per case: $580 / 350 = $1.66/case

Compare with other departments:
  PKG01: $1.58/case (more efficient)
  QUALITY: $2.00/case (low volume, high cost per case)
```

### Department ← Equipment Maintenance (Scheduling)

```
Preventive Maintenance by Department:

Department GB01 Maintenance Schedule:
  Scale 1: Calibration every 30 days (next: 2026-04-10)
  Scale 2: Calibration every 30 days (next: 2026-04-10)
  Scale 3: Calibration every 30 days (next: 2026-04-10)
  Conveyor: Inspection every 7 days
  Printer: Supplies check every 3 days

Department PKG01:
  Scale 4: Calibration every 30 days (next: 2026-04-08)
  Scale 5: Calibration every 30 days (next: 2026-04-12)
  Conveyor: Inspection every 7 days
  Printer: Supplies check weekly

Department QUALITY:
  Scale 6: Calibration every 30 days (next: 2026-04-15)
  Printer: Supplies check monthly

Enables: Equipment tracking, downtime prediction, spare parts planning
```

### Department ← Shift Coverage (Staffing)

```
Staffing by Department & Shift:

Shift 1 (6 AM - 2 PM):
  Department GB01: jsmith, rrodriguez (2 operators)
  Department PKG01: akim, cchen (2 operators)
  Department QUALITY: manager01 (1 operator, also supervises)

Shift 2 (2 PM - 10 PM):
  Department GB01: mgarcia, jsanchez (2 operators)
  Department PKG01: mgarcia, jsanchez (shared with GB01)
  Department QUALITY: manager01 (1 operator, also supervises)

Shift 3 (10 PM - 6 AM):
  Department GB01: jsmith, rrodriguez (on-call, cross-trained)
  Department PKG01: mgarcia, jsanchez (on-call)
  Department QUALITY: Automated checks only

Enables: Shift scheduling, coverage planning, cross-training identification
```

---

## 7. Department Workflows

### Workflow 1: Product Through Multiple Departments

```
Product enters facility → Goes through departments in sequence

1. RECEIVING (Department RCV01)
   - Input: Bulk meat or raw ingredient shipment
   - Process: Unload, initial inspection
   - Output: Meat staged for processing
   - Scales: None

2. PROCESSING (Department GB01 - Ground Beef)
   - Input: Bulk meat from Receiving
   - Process: Grinding, seasoning, mixing
   - Output: Portioned ground beef
   - Scales: 1, 2, 3
   - Quality Check: Temperature, visual

3. PACKAGING (Department PKG01)
   - Input: Ground beef from Processing
   - Process: Place in containers, seal, weigh
   - Output: Packaged cases
   - Scales: 4, 5
   - Quality Check: Weight verification, seal integrity

4. QUALITY ASSURANCE (Department QUALITY)
   - Input: Packaged cases from Packaging
   - Process: Final inspection, defect check
   - Output: Approved or rejected cases
   - Scales: 6
   - Quality Check: Final weight verification

5. SHIPPING (Department SHIP01)
   - Input: Approved cases from QA
   - Process: Consolidate, palletize, load truck
   - Output: Shipment-ready pallets
   - Scales: None (pallet weight only)

Audit Trail (by Serial):
  Serial ABC-001
    - Processed in GB01 (Scale 2): 06:30 AM
    - Moved to PKG01 (Scale 4): 08:45 AM
    - Moved to QUALITY (Scale 6): 10:15 AM
    - Shipped in SHIP01: 15:30 PM
    - Total time in system: 9 hours
    - Total time on hold in PKG01: 1.5 hours (bottleneck identified)
```

### Workflow 2: Department Capacity Management

```
Night Before Planning:

Orders for tomorrow:
  - Customer A: 2,000 cases of Product 1001
  - Customer B: 1,500 cases of Product 1002
  - Customer C: 1,000 cases of Product 1003
  Total: 4,500 cases needed

Facility Capacity Review:
  Department GB01: 500 cases/hour × 8 hours = 4,000 cases/shift
  Department PKG01: 400 cases/hour × 8 hours = 3,200 cases/shift ← BOTTLENECK
  Department QUALITY: 300 cases/hour × 8 hours = 2,400 cases/shift

Problem: PKG01 is bottleneck, can only handle 3,200 cases

Solutions:
  Option 1: Run PKG01 over two shifts (16 hours) = 6,400 cases capacity
  Option 2: Reduce order intake to 3,200 cases
  Option 3: Process 4,500 cases with overtime/double shift in PKG01

Decision: Run PKG01 on double shift
  - Shift 1: Operators A, B (6 AM - 2 PM)
  - Shift 2: Operators C, D (2 PM - 10 PM)
  - Both shifts: 400 cases/hour × 8 hours × 2 = 6,400 cases total

Result:
  GB01: Produces 4,000 cases on Shift 1 (handles input)
  PKG01: Processes 4,000 cases across Shift 1 & 2
  QUALITY: Processes 4,000 cases across Shift 1 & 2 (capacity adequate)
  All customers satisfied, no backlog
```

---

## 8. Department Configuration Examples

### Example 1: Small Single-Line Facility

**Setup:** One production line, all steps in one "department"

```
Facility: SMALL_PLANT_NJ

Department: PRODUCTION
  ├─ Location: Building A, All Floors
  ├─ Capacity: 200 cases/hour
  ├─ Scales:
  │   ├─ Scale 1: Intake weighing
  │   └─ Scale 2: Final pack weight
  ├─ Printer: LBL-001
  ├─ Operators:
  │   ├─ Shift 1: jsmith, rrodriguez
  │   ├─ Shift 2: mgarcia, akim
  │   └─ Shift 3: jsanchez (cross-trained)
  └─ Products: All product types

Production Flow:
  Input → PRODUCTION (Scale 1, 2) → Output (all in one dept)
  Simple workflow, minimal handoffs, single quality gate
```

---

### Example 2: Medium Multi-Line Facility

**Setup:** Multiple production lines with dedicated departments

```
Facility: CHICAGO_PLANT

Department: GB01 (Ground Beef Line 1)
  ├─ Building A, Floor 1
  ├─ Capacity: 400 cases/hour
  ├─ Scales: 1, 2, 3
  ├─ Operators: jsmith, rrodriguez (Shift 1)
  ├─ Printer: LBL-001
  └─ Products: Ground Beef products

Department: GB02 (Ground Beef Line 2)
  ├─ Building A, Floor 1
  ├─ Capacity: 400 cases/hour
  ├─ Scales: 4, 5, 6
  ├─ Operators: mgarcia, akim (Shift 2)
  ├─ Printer: LBL-002
  └─ Products: Ground Beef products

Department: PKG01 (Packaging Line)
  ├─ Building B, Floor 1
  ├─ Capacity: 600 cases/hour
  ├─ Scales: 7, 8
  ├─ Operators: cchen, jsanchez (Shift 1)
  ├─ Printer: LBL-003
  └─ Products: All packaged products

Department: QUALITY (QC Department)
  ├─ Building C, Floor 1
  ├─ Capacity: 400 cases/hour
  ├─ Scales: 9
  ├─ Operators: manager01 (All shifts)
  ├─ Printer: LBL-004
  └─ Products: All products

Department: SHIP01 (Shipping)
  ├─ Building C, Floor 2
  ├─ Capacity: 2,000 cases/day
  ├─ Scales: None (pallet weight only)
  ├─ Operators: shipping_team
  └─ Products: All products

Production Flow:
  GB01 (400 CPH) ──┐
                   ├──→ PKG01 (600 CPH) ──→ QUALITY (400 CPH) ──→ SHIP01 (2000/day)
  GB02 (400 CPH) ──┘

Bottleneck: QUALITY at 400 CPH (two lines feeding 600 cases/hour input)
Solution: Run PKG01 output on two quality shifts or add equipment
```

---

### Example 3: Large Multi-Facility Meat Processing

**Setup:** Multiple plants, each with multiple departments

```
Company: NATIONAL_MEAT_CO

FACILITY: MIAMI (Beef Processing)
  Department: GB01 (Ground Processing)
  Department: PKG01 (Primary Packaging)
  Department: QUALITY (Final QC)
  Department: SHIP01 (Shipping)
  Total Capacity: ~350 CPH (bottleneck at PKG01: 300 CPH)

FACILITY: CHICAGO (Poultry Processing)
  Department: CH01 (Chicken Processing)
  Department: CH02 (Deboning)
  Department: PKG01 (Packaging)
  Department: QUALITY (Final QC)
  Department: SHIP01 (Shipping)
  Total Capacity: ~500 CPH (bottleneck at CH02: 400 CPH)

FACILITY: IOWA (Pork Processing)
  Department: PK01 (Pork Processing)
  Department: PK02 (Curing)
  Department: PKG01 (Packaging)
  Department: QUALITY (Final QC)
  Department: SHIP01 (Shipping)
  Total Capacity: ~450 CPH (balanced across line)

Corporate Dashboard:
  ┌──────────────────────────────────────┐
  │ National Production Summary            │
  ├──────────────────────────────────────┤
  │ Miami:   1,250 cases (capacity 80%)   │
  │ Chicago: 2,100 cases (capacity 95%)   │
  │ Iowa:    1,800 cases (capacity 85%)   │
  │ Total:   5,150 cases/day              │
  └──────────────────────────────────────┘

Bottleneck Analysis:
  Chicago - CH02 (Deboning) at critical capacity
  Action: Approve temporary labor increase or overtime

Consolidation Report:
  Total GB (Miami): 1,250 cases
  Total CH (Chicago): 2,100 cases
  Total PK (Iowa): 1,800 cases
  Company Total: 5,150 cases/day
```

---

## 9. Common Questions & Troubleshooting

### Q: How do we track which department produced a specific case?

**A:** Through the Scale and Serial relationship:

```sql
/* To find which department produced Serial ABC-001 */
SELECT 
    sr.SerialNum,
    s.ScaleNumber,
    s.Department,
    s.ScaleDescription,
    sr.PrintTime,
    sr.Operator
FROM Serial sr
JOIN Scale s ON sr.ScaleNumber = s.ScaleNumber
WHERE sr.SerialNum = 'ABC-001';

Result:
SerialNum  ScaleNumber  Department  Description           PrintTime  Operator
ABC-001    2            GB01        GB-Scale-C (Final)    06:45:30   jsmith
```

### Q: What if we need to move a scale to a different department?

**A:** Update Scale.Department assignment:

```sql
/* Move Scale 5 from PKG01 to PKG02 */
UPDATE Scale
SET Department = 'PKG02'
WHERE ScaleNumber = 5;

Effect:
  - All future Serials created at Scale 5 will show Department = PKG02
  - Historical Serials (that used Scale 5 before) still show old department
  - May cause inconsistencies in historical reports - document the change
```

**Best Practice:**

```
1. Document the move with reason and timestamp
2. Note: Historical data affected (before/after date)
3. Run reconciliation report: all serials by scale/department
4. Update department staffing assignments if operators moved
5. Update printer assignments if changed
6. Update capacity calculations if throughput changed
7. Notify all affected systems (reporting, forecasting)
```

### Q: How do we manage departments in a multi-shift environment?

**A:** Department structure remains same, operators rotate:

```
Department GB01 (operates 24/7 across three shifts)

Shift 1 (6 AM - 2 PM):
  Operators: jsmith, rrodriguez
  Machines: Operational 100%
  Production: 400 cases/hour
  Quality: Full QC procedures

Shift 2 (2 PM - 10 PM):
  Operators: mgarcia, akim
  Machines: Operational 100%
  Production: 400 cases/hour
  Quality: Full QC procedures

Shift 3 (10 PM - 6 AM):
  Operators: Low staffing (2 operators shared with other depts)
  Machines: Operational at 80% (maintenance window 2 AM - 4 AM)
  Production: 300 cases/hour
  Quality: Automated checks + spot verification

Daily: GB01 produces (400 + 400 + 300) = 1,100 cases
```

### Q: How do we calculate department efficiency?

**A:** Compare actual vs. capacity:

```
Department GB01 Efficiency Report (Date: 03/11/2026)

Theoretical Capacity:
  500 cases/hour × 8 hours = 4,000 cases maximum

Actual Production:
  350 cases/hour average × 8 hours = 2,800 cases actual

Efficiency:
  2,800 / 4,000 = 70% efficiency

Downtime Analysis:
  - Scheduled maintenance: 1 hour (400 cases)
  - Equipment breakdown: 0.5 hours (250 cases)
  - Material shortage: 0.5 hours (250 cases)
  - Total downtime: 2 hours (900 cases)
  - Personnel breaks: 0.5 hours (250 cases)

Explanation: 4,000 theoretical - 900 downtime - 250 breaks = 2,850 net capacity
Actual 2,800 vs. net 2,850 = 98% utilization of available time
```

### Q: Can a serial belong to multiple departments?

**A:** No, each serial belongs to exactly ONE department (via its scale).

However, a case may be PROCESSED through multiple departments:

```
Serial ABC-001:
  ├─ Created at Scale 2 (Department GB01) → Initially Department = GB01
  ├─ Moved to Department PKG01 for next production step
  ├─ Moved to Department QUALITY for verification
  ├─ Moved to Department SHIP01 for shipment
  
But Serial.Department field = GB01 (where originally created/weighed)

Transit tracking through other departments is managed via:
  - Process flow documentation
  - Audit trail / location tracking
  - Movement records (if system supports multi-dept tracking)
  - Time stamps showing when cases moved between areas
```

### Q: What if a department needs to be closed or consolidated?

**A:** Plan closure and migrate equipment:

```
Department GB01 (closing 2026-06-30)
  Scales: 1, 2, 3 → Reassign to GB02

Steps:
1. Notify all operators of department closure
2. Retrain operators to GB02 (consolidate lines)
3. Move scales 1, 2, 3 physical equipment to GB02
4. Update Scale.Department for scales 1, 2, 3 to "GB02"
5. Update capacity calculations:
   - GB02 capacity increases from 400 to 900 CPH
6. Create audit log documenting:
   - Closure date: 2026-06-30
   - Reason: Line consolidation for efficiency
   - Serials affected: Last 5 created in GB01 (dates, serials)
7. Archive GB01 department record (mark inactive)
8. Run historical report: All GB01 production before closure

Result:
  - All GB01 future production → Assigned to GB02
  - Historical GB01 data preserved with old department ID
  - Reports can filter by original department or consolidated view
```

---

## 10. Department Performance Analytics

### Production by Department (Daily)

```
Daily Production Report - 2026-03-11

Department  Cases    TotalWgt   AvgWgt   Efficiency  Status
GB01        350      2,842.50   8.12     70%         ✓ Normal
GB02        380      3,084.00   8.11     76%         ✓ Normal
PKG01       395      3,207.50   8.12     79%         ✓ Normal
QUALITY     385      3,128.75   8.13     96%         ✓ High
SHIP01      380      3,084.00   8.12     N/A         ✓ Normal

System Total: 1,890 cases
Bottleneck: PKG01 (limiting factor at 79%)
Recommended Action: Increase staffing or schedule second shift
```

### Equipment Utilization by Department

```
Equipment Status Report - 2026-03-11 (Shift 1)

Department  Scale  Status       Uptime  Cases    CPH
GB01        1      OPERATIONAL  100%    85       170
GB01        2      OPERATIONAL  95%     160      320
GB01        3      DOWN (repair) 0%      0        0
            Subtotal:           65%     245      490

GB02        4      OPERATIONAL  100%    95       190
GB02        5      OPERATIONAL  98%     165      330
            Subtotal:           99%     260      520

PKG01       6      OPERATIONAL  100%    120      240
PKG01       7      OPERATIONAL  100%    145      290
            Subtotal:           100%    265      530

QUALITY     8      OPERATIONAL  100%    125      250
            Subtotal:           100%    125      250

Facility Total Production: 895 cases in first 4 hours
Projected Daily (8 hr shift): 1,790 cases
Status: GB01 Scale 3 offline - reducing throughput by 10%
ACTION ITEM: Repair Scale 3 to restore full GB01 capacity
```

---

## 11. Department vs. Other Concepts

### Department vs. Shift (DIFFERENT)

| Aspect | Department | Shift |
| --- | --- | --- |
| **Definition** | Physical production area/equipment | Time period when work occurs |
| **Change Frequency** | Rarely (facility structure) | Daily/recurring (scheduling) |
| **Example** | GB01, PKG01, QUALITY | Shift 1 (6 AM-2 PM), Shift 2 (2 PM-10 PM) |
| **Captures** | WHERE production happened | WHEN production happened |

**Combined Example:**

```
Serial ABC-001
  ├─ Department = "GB01" (WHERE - ground beef line)
  ├─ Shift = 1 (WHEN - morning shift)
  ├─ Operator = "jsmith" (WHO)
  └─ ScaleNumber = 2 (WHAT equipment)

Enables complete traceability:
  "Who (jsmith) produced what (Serial ABC-001)
   where (Department GB01) when (Shift 1)"
```

### Department vs. Operator (DIFFERENT)

| Aspect | Department | Operator |
| --- | --- | --- |
| **Definition** | Physical production area | Individual person |
| **Multiplicity** | Fixed (facility structure) | Variable (staffing levels) |
| **Example** | Department GB01 | Operator jsmith |
| **Assignment** | Equipment permanently in dept | Operators rotate through depts |

**Example:**

```
Department GB01 has scales 1, 2, 3 (permanent)
Multiple operators work in GB01 (rotating):
  Shift 1: jsmith, rrodriguez
  Shift 2: mgarcia, akim
  Shift 3: jsanchez (on-call)

Each operator can work in multiple departments over time
Each department serves same role regardless of operator
```

---



---

## 13. Department Configuration Checklist

### Setting Up Departments in New Facility

- [ ] Identify all production lines/functional areas
- [ ] Assign Department IDs (codes and names)
- [ ] Map physical facility locations to departments
- [ ] Assign scales to departments (StartScale/EndScale or explicit)
- [ ] Assign printers to each department
- [ ] Define department capacity limits (cases/hour)
- [ ] Define minimum/maximum staffing per department
- [ ] Assign initial operators to departments/shifts
- [ ] Document department workflows (product flow)
- [ ] Set up maintenance schedules per department
- [ ] Create department-specific QC procedures
- [ ] Configure cost allocation for each department
- [ ] Test department filtering in production reports
- [ ] Train staff on new department structure
- [ ] Monitor first week for capacity/bottleneck issues
- [ ] Document any changes or modifications

### Ongoing Department Management

- [ ] Monthly efficiency reports by department
- [ ] Quarterly capacity planning review
- [ ] Equipment maintenance tracking per department
- [ ] Staff scheduling updates per department
- [ ] Cross-training documentation (operators across depts)
- [ ] Safety incident tracking by department
- [ ] Quality metrics monitoring per department
- [ ] Bottleneck analysis and mitigation
- [ ] Cost analysis by department (labor, equipment, overhead)

---

## 14. Summary

| Aspect | Details |
| --- | --- |
| **Purpose** | Organize production into functional areas, track department-level metrics |
| **Definition** | Logical grouping of scales and operators for production work |
| **Structure** | Implicit (via Scale assignments, not separate master table) |
| **Primary Link** | Scale assignment (scales belong to departments) |
| **Serial Tracking** | Implicit via scale → department relationship |
| **Capacity Mgmt** | Monitor throughput by department to identify bottlenecks |
| **Key Relationships** | Scale (equipment), Operator (labor), Serial (production), Goals (targets), Shift (timing) |
| **Performance** | Measure efficiency, identify bottlenecks, optimize workflow |
| **Configuration** | Implicit department assignment to scales |
| **Scalability** | Supports single-line small plants to multi-facility large organizations |
| **Quality** | Department-specific QC gates at each production stage |
| **Compliance** | Enables location-level traceability for regulatory requirements |

---

