# Serial: Complete Concept Guide

## Overview

**Serial** is the **master production record** in the legacy SLC application. Each Serial record represents **one individual case, box, or unit that was manufactured** and tracked from production through shipment.

Think of Serial as a **digital "label tag"** that follows a physical unit through the entire production lifecycle:

- When printed with a barcode/label
- Weight measurements captured
- Dates assigned (print, pack, kill dates)
- Linked to production goals
- Packed into pallets
- Shipped to customers

**Serial records are the foundation of traceability** - every unit of production has a corresponding Serial record.

---

## Why Serial Exists

### Problems It Solves

1. **Traceability** - Track every individual unit from production to shipment
2. **Quality Tracking** - Link units to batches, lots, and operators for quality investigations
3. **Weight Compliance** - Record actual weights for regulatory compliance (FDA, USDA)
4. **Inventory Control** - Know exact quantities produced and location
5. **Goal Achievement** - Measure progress toward production targets (goals)
6. **Recalls** - Quickly identify affected units by lot, date, or serial
7. **Customer Service** - Answer "Who made this? When? Where?"

### Business Value

✅ **Regulatory Compliance** - Prove production dates, weights, lot numbers  
✅ **Recall Management** - Rapidly identify and isolate affected units  
✅ **Quality Management** - Correlate issues to batches/operators/shifts  
✅ **ERP Integration** - Send production data to enterprise systems  
✅ **Performance Metrics** - Measure yield, efficiency, by shift/operator  
✅ **Customer Confidence** - Full traceability for food safety requirements

---

## Table Schema

### Serial Table

**Primary Key:** `SerialNum` + `AddOrDel`

```sql
CREATE TABLE Serial (
    -- Identification
    SerialNum           CHAR(12) NOT NULL,        -- PK1: Unique serial/barcode
    AddOrDel            CHAR(1) NOT NULL,         -- PK2: 'A'=Add, 'D'=Delete/Cancel
    ProductCode         INT NOT NULL,             -- FK: Product being made
    ItemCode            INT DEFAULT 0,            -- Item code if multi-item

    -- Timing
    PrintDate           DATE,                     -- When label was printed
    PrintTime           INT DEFAULT 0,            -- Time label printed (seconds)
    PackDate            DATE,                     -- When product was packed/produced
    KillDate            DATE,                     -- Product expiration/kill date

    -- Production Context
    ScaleNumber         INT DEFAULT 0,            -- Which scale produced it (1-99)
    Shift               INT DEFAULT 1,            -- Which shift (1-3)
    Operator            CHAR(10),                 -- Who printed the label
    GoalID              INT DEFAULT 0,            -- FK: Production goal this contributes to

    -- Weights
    LabelWgt            DECIMAL DEFAULT 0,        -- Weight printed on label
    NetWgt              DECIMAL DEFAULT 0,        -- Actual net product weight
    BoxTare             DECIMAL DEFAULT 0,        -- Packaging weight (tare)
    ExtraTare           DECIMAL DEFAULT 0,        -- Additional tare (moisture, etc.)
    ContainerPreTareWgt DECIMAL DEFAULT 0,        -- Pre-tare before filling
    ItemPreTareWgt      DECIMAL DEFAULT 0,        -- Item container tare
    ItemPostTareWgt     DECIMAL DEFAULT 0,        -- Post-filling item tare
    CasePostTareWgt     DECIMAL DEFAULT 0,        -- Post-filling case tare
    TotalTareWgt        DECIMAL DEFAULT 0,        -- Sum of all tares

    -- Offsets & Dates
    SellByOffset        INT DEFAULT 0,            -- Days from KillDate to SellBy
    VarOffset           INT DEFAULT 0,            -- Variable date offset

    -- Product Data
    Lot                 CHAR(10) DEFAULT "000",   -- Lot/batch number
    Price               DECIMAL DEFAULT 0,        -- Unit price
    CustomerName        CHAR(20),                 -- End customer name
    Order               CHAR(20),                 -- Customer order number

    -- Barcodes & Labels
    UserBarString       CHAR(32),                 -- Custom barcode value
    ItemWgtList         CHAR(40),                 -- CSV of item weights

    -- Status & Tracking
    Sent                LOGICAL DEFAULT NO,       -- Shipped to ERP/customer system?
    Reprinted           LOGICAL DEFAULT NO,       -- Was label reprinted?
    ProducedAtVRT       LOGICAL DEFAULT NO,       -- From VRT (variable rate) system?

    -- Inventory/Order Links
    PalletNum           CHAR(10),                 -- Which pallet (PlantID+YDDD+seq)
    PdnOrdNum           CHAR(10),                 -- Production order number
    PdnOrdLineNum       INT DEFAULT 0,            -- PO line number
    MfgOrdNum           INT64 DEFAULT 0,          -- Manufacturing order number
    PDNBatchID          INT64 DEFAULT 0,          -- Batch ID from PDN system

    -- Other
    WgtUnits            CHAR(2) DEFAULT "LB"      -- "LB" or "KG"
);

-- Primary Key Index (composit)
CREATE UNIQUE INDEX PK_Serial ON Serial(SerialNum, AddOrDel);

-- Secondary Indexes for queries
CREATE INDEX IX_Serial_ProductCode ON Serial(ProductCode);
CREATE INDEX IX_Serial_GoalID ON Serial(GoalID);
CREATE INDEX IX_Serial_PrintDate ON Serial(PrintDate);
CREATE INDEX IX_Serial_KillDate ON Serial(KillDate);
CREATE INDEX IX_Serial_Sent ON Serial(Sent);
CREATE INDEX IX_Serial_Shift ON Serial(Shift);
CREATE INDEX IX_Serial_PalletNum ON Serial(PalletNum);
```

---

## Field Descriptions: Complete Reference

### Identification Fields

| Field         | Type        | Purpose                            | Example                        |
| ------------- | ----------- | ---------------------------------- | ------------------------------ |
| **SerialNum** | CHAR(12) PK | Unique identifier for this unit    | "ABC-123-456", "SN20260311001" |
| **AddOrDel**  | CHAR(1) PK  | 'A'=Created, 'D'=Deleted/Cancelled | "A"                            |

### Product & Item Reference

| Field           | Type   | Purpose                       | Example                |
| --------------- | ------ | ----------------------------- | ---------------------- |
| **ProductCode** | INT FK | What product was made         | 1001 (Milk 2L)         |
| **ItemCode**    | INT    | If part of multi-item product | 5021 (Individual item) |

### Timing Fields (Most Critical)

| Field         | Type | Purpose                          | Relationship                       |
| ------------- | ---- | -------------------------------- | ---------------------------------- |
| **PrintDate** | DATE | When barcode label was printed   | First timestamp in lifecycle       |
| **PrintTime** | INT  | Seconds of day label printed     | Precise per-second timing          |
| **PackDate**  | DATE | When product was actually packed | When made/produced                 |
| **KillDate**  | DATE | Product expiration date          | Basis for SellByOffset calculation |

**Timing Relationship:**

```
PrintDate/PrintTime
    ↓ (label created)
PackDate (production date)
    ↓ (PackDate + SellByOffset days)
KillDate (expiration)
    ↓ (derived from KillDate + SellByOffset)
SellByDate (displayed on label)
```

### Production Context

| Field           | Type     | Purpose                            | Notes                          |
| --------------- | -------- | ---------------------------------- | ------------------------------ |
| **ScaleNumber** | INT      | Which scale produced it            | 1-99 (physical scale ID)       |
| **Shift**       | INT      | Which shift (1, 2, or 3)           | Links to shift-specific config |
| **Operator**    | CHAR(10) | Who printed the label              | Audit trail, accountability    |
| **GoalID**      | INT FK   | Production goal this helps achieve | May be 0 if not under goal     |

### Weight Fields (Complex)

```
LabelWgt        ← Weight PRINTED on label
       ↓
NetWgt          ← Actual product weight (actual measure)
       ↓
       = (NetWgt + Tares) should = LabelWgt (or within tolerance)

Tare Components:
  BoxTare              ← Packaging container
  ExtraTare            ← Moisture, ice, etc.
  ContainerPreTareWgt  ← Container before filling
  ItemPreTareWgt       ← Item container before filling
  ItemPostTareWgt      ← Item container after filling
  CasePostTareWgt      ← Case container after filling
  TotalTareWgt         ← Sum of all above
```

| Field                   | Purpose                               | Example  |
| ----------------------- | ------------------------------------- | -------- |
| **LabelWgt**            | Weight printed on the barcode/label   | 2.50 lbs |
| **NetWgt**              | Actual product weight (contents only) | 2.30 lbs |
| **BoxTare**             | Packaging weight                      | 0.20 lbs |
| **ExtraTare**           | Additional tare (moisture, ice)       | 0.05 lbs |
| **ContainerPreTareWgt** | Container before filling              | 0.15 lbs |
| **TotalTareWgt**        | Sum of all tare values                | 0.40 lbs |

**Weight Validation Logic:**

```
Expected: NetWgt + TotalTareWgt = LabelWgt
Actual:   2.30 + 0.40 = 2.70 lbs (printed weight)

If difference > tolerance: Weight error/warning
```

### Offset Fields (Date Calculations)

| Field            | Purpose                           | Example                        |
| ---------------- | --------------------------------- | ------------------------------ |
| **SellByOffset** | Days from KillDate to SellBy date | 7 (SellBy = KillDate + 7 days) |
| **VarOffset**    | Variable offset from today        | -2 (2 days before today)       |

### Lot & Pricing

| Field     | Purpose                          | Example     | Notes                       |
| --------- | -------------------------------- | ----------- | --------------------------- |
| **Lot**   | Lot/batch identifier             | "20260311A" | Alpha-numeric, max 10 chars |
| **Price** | Unit price at time of production | 4.99        | Per unit/lb/case            |

### Customer Information

| Field            | Purpose                    | Example                  |
| ---------------- | -------------------------- | ------------------------ |
| **CustomerName** | End customer/retailer name | "Walmart", "Whole Foods" |
| **Order**        | Customer PO number         | "PO-2026-45123"          |

### Barcode & Label

| Field             | Purpose                  | Example                          | Max Length |
| ----------------- | ------------------------ | -------------------------------- | ---------- |
| **UserBarString** | Custom barcode value     | "ABC123\|Price4.99\|Lot20260311" | 32 chars   |
| **ItemWgtList**   | CSV of weights for items | "1.2,2.1,0.8"                    | 40 chars   |

**UserBarString Format:**
Often stores name-value pairs (NVP) separated by pipe `|`:

```
ZuluDept=PackingDept | LotNum=20260311 | Grade=Prime
```

### Status Tracking

| Field             | Type    | Purpose                            | Values |
| ----------------- | ------- | ---------------------------------- | ------ |
| **Sent**          | LOGICAL | Sent to ERP/customer system        | YES/NO |
| **Reprinted**     | LOGICAL | Label was reprinted (not original) | YES/NO |
| **ProducedAtVRT** | LOGICAL | Produced on variable rate system   | YES/NO |

### Pallet & Order Links

| Field             | Purpose                  | Format       | Notes                        |
| ----------------- | ------------------------ | ------------ | ---------------------------- |
| **PalletNum**     | Which pallet packed into | "99YDDD9999" | PlantID + Yr-Day + seq       |
| **PdnOrdNum**     | Production order         | "X(10)"      | Links to manufacturing order |
| **PdnOrdLineNum** | PO line number           | "X(10)"      | Which line on the order      |
| **MfgOrdNum**     | Manufacturing order      | INT64        | From manufacturing system    |
| **PDNBatchID**    | Batch from PDN system    | INT64        | External batch tracking      |

### Other

| Field        | Purpose      | Values     |
| ------------ | ------------ | ---------- |
| **WgtUnits** | Weight units | "LB", "KG" |

---

## Related Concept: ItemSerial

**ItemSerial** is to **items** what **Serial** is to **cases**.

When a case contains multiple items (6-pack, multi-pack), each item gets its own ItemSerial record:

```
Serial Record:
  SerialNum = "ABC-123-456"
  ProductCode = 3100
  LabelWgt = 5.0 lbs (total case)
  NetWgt = 4.5 lbs (total case)

ItemSerial Records (multiple):
  ItemSerial 1: ItemCode=5021, NetWgt=0.75 lbs
  ItemSerial 2: ItemCode=5021, NetWgt=0.75 lbs
  ItemSerial 3: ItemCode=5021, NetWgt=0.75 lbs
  ItemSerial 4: ItemCode=5021, NetWgt=0.75 lbs
  ItemSerial 5: ItemCode=5021, NetWgt=0.75 lbs
  ItemSerial 6: ItemCode=5021, NetWgt=0.75 lbs
  Total: 4.5 lbs ✓
```

**ItemSerial fields are similar to Serial**, with additions:

- **ItemCode** - Which item produced
- **CaseSerialNum** - Links back to parent Serial
- **ItemPostTareWgt** - Item-specific tare
- **ExtraTareWgt** - Item-specific extra tare

---

## Relationships to Other Concepts

### 1. Serial ← Product (Many-to-One)

```
Product (1:N) ──→ Serial

One Product can have MANY Serial records
Each Serial has exactly ONE Product

Example:
  Product 1001 (Milk 2L)
    → Serial ABC-123-001
    → Serial ABC-123-002
    → Serial ABC-123-003
    ... (thousands of serials)
```

**Query:** Find all units of Product 1001 produced today:

```sql
SELECT * FROM Serial
WHERE ProductCode = 1001
  AND PrintDate = TODAY
ORDER BY SerialNum;
```

### 2. Serial ← Goals (Many-to-One)

```
Goals (1:N) ──→ Serial

One Goal can have MANY Serial records
Each Serial may (or may not) contribute to a Goal

Example:
  Goal ID 12345: "Produce 1000 cases by 2:00 PM"
    → Serial ABC-123-001 (counts toward goal)
    → Serial ABC-123-002 (counts toward goal)
    → Serial ABC-123-003 (counts toward goal)
    ...
    → Serial BAD-456-789 (GoalID=0, off-goal production)
```

**Measurement:**

- Goal specifies: `TargetLevel = 1000` (count)
- System counts: `COUNT(Serial WHERE GoalID=12345)`
- When count reaches 1000, goal is achieved

### 3. Serial ← Operator (Many-to-One)

```
Operator (1:N) ──→ Serial

One Operator can produce MANY Serial records
Shows accountability and performance

Example:
  Operator "jsmith" (John Smith)
    → Printed label for Serial ABC-123-001
    → Printed label for Serial ABC-123-002
    → Printed label for Serial ABC-123-003
    ...
```

**Quality Investigation:** "Product quality issue found - find all units printed by jsmith shifts 1-2 on 3/1"

```sql
SELECT * FROM Serial
WHERE Operator = 'jsmith'
  AND Shift IN (1, 2)
  AND PrintDate = '03/01/2026'
ORDER BY PrintTime;
```

### 4. Serial → PalletDtl (One-to-Many)

```
Serial (1:N) ──→ PalletDtl

One Serial can be packed into ONE pallet detail record
Tracks physical packing

Example:
  Serial ABC-123-001
    → Packed into PalletNum "99YDDD0001" at position 1
    → Weight on pallet: 2.5 lbs
```

**Query:** Find all serials packed into pallet:

```sql
SELECT s.SerialNum, s.NetWgt, pd.LabelWgt
FROM Serial s
JOIN PalletDtl pd ON s.SerialNum = pd.SerialNum
WHERE pd.PalletNum = '99YDDD0001'
ORDER BY s.SerialNum;
```

### 5. Serial ← Totals (Aggregation)

```
Serial records ──aggregates to──→ Totals

Totals table sums up Serial production:
  SELECT SUM(COUNT(*)) FROM Serial
  GROUP BY ProductCode, Shift, Scale, PrintDate

Example:
  Totals record:
    ProductCode = 1001
    Scale = 1
    Shift = 1
    PrintDate = 03/11/2026
    TotalCount = 150 (150 serials with these filters)
    TotalNetWgt = 375.5 lbs (sum of NetWgt)
    TotalLabelWgt = 390.0 lbs (sum of LabelWgt)
```

### 6. Serial → BatchOrder (Optional Link)

```
Serial can link to BatchOrder:
  Serial.Order ← BatchOrder.OrdNum

Groups multiple serials under a batch/order
```

### 7. Serial ← ItemSerial (One-to-Many)

```
Serial (1:N) ──→ ItemSerial

One Serial (case) can contain MANY ItemSerial records (items)

Example:
  Serial ABC-123-456 (6-pack case)
    → ItemSerial: Item 1 (0.75 lbs)
    → ItemSerial: Item 2 (0.75 lbs)
    → ItemSerial: Item 3 (0.75 lbs)
    → ItemSerial: Item 4 (0.75 lbs)
    → ItemSerial: Item 5 (0.75 lbs)
    → ItemSerial: Item 6 (0.75 lbs)
    Total: 4.5 lbs ✓
```

---

## Serial Lifecycle: Complete Flow

### Phase 1: Creation (At Label Print)

```
Timeline:
  1. Operator scans product barcode
  2. System loads Product record with ProcessSequence
  3. System executes ProductProcess:
     a. CasePrint process: Generate label
     b. System creates NEW Serial record:
        SerialNum = Generate unique 12-char barcode
        ProductCode = [scanned product]
        AddOrDel = "A" (Added)
        PrintDate = TODAY
        PrintTime = SECONDS-OF-SECOND
        Operator = [current user]
        Shift = [current shift]
        ScaleNumber = [current scale]
     c. Default weights from Product record
     d. Send to printer

  Serial State: CREATED, NOT YET WEIGHED
```

### Phase 2: Weighing (At Scale)

```
Timeline:
  1. Operator places case on scale
  2. Scale reads weight
  3. System updates Serial:
     LabelWgt = [weight from scale]
     NetWgt = [weight calculation]
     BoxTare = [from Product record]
     TotalTareWgt = [calculated]

  4. Weight validation:
     IF |NetWgt - ExpectedNetWgt| > Tolerance THEN
         Show warning to operator
     END.

  Serial State: WEIGHED, READY FOR OUTPUT
```

### Phase 3: Additional Processing

```
Depending on ProductProcess:

ItemPrint:
  - Create ItemSerial records for each item
  - Link via CaseSerialNum

VariableData:
  - Update KillDate, SellByDate
  - Apply offsets (SellByOffset, VarOffset)
  - Update custom fields

PalletOutput:
  - Create PalletHdr record
  - Create PalletDtl record linking Serial→Pallet
```

### Phase 4: Goal Completion (Optional)

```
If Serial is linked to a Goal:
  1. GoalID is populated
  2. System increments goal count
  3. IF count >= TargetLevel THEN
       Goal Status = "Achieved"
       Record timestamp
       Notify supervisor
     END.
```

### Phase 5: Cancellation (AddOrDel = "D")

```
If unit needs to be removed:
  1. Create new Serial record with:
     SerialNum = [original serial]
     AddOrDel = "D"  (Deleted/Cancelled)
  2. Original record remains for audit trail
  3. Subtracted from goal count
  4. Removed from pallet

Serial State: LOGICALLY DELETED (audit trail preserved)
```

### Phase 6: Shipment/Sending

```
When unit is shipped:
  1. System aggregates serials by order/customer
  2. Exports to ERP/customer system
  3. Updates Serial.Sent = YES
  4. Records timestamp

Serial State: SENT, SHIPPED
```

---

## Primary Key: SerialNum + AddOrDel

The composite key `(SerialNum, AddOrDel)` is crucial:

```
SerialNum = "ABC-123-456"

Record 1: (SerialNum="ABC-123-456", AddOrDel="A")
  ← Original creation
  ← All data for this unit

Record 2: (SerialNum="ABC-123-456", AddOrDel="D")
  ← Cancellation/deletion marker
  ← Used to subtract from counts/totals
  ← Original record 1 remains (audit trail)

Query: Find net production of Serial ABC-123-456
  SELECT * FROM Serial
  WHERE SerialNum = "ABC-123-456"
  ORDER BY AddOrDel

  Result: 2 rows - shows original (A) and cancellation (D)
```

**Business Logic:**

```
Effective Production = Sum of [A records] - Count of [D records]
```

---

## Indexes and Query Optimization

### Primary Index

```
UNIQUE (SerialNum, AddOrDel)
  - Ensures one create + one delete per serial
  - Used for lookups: SELECT * WHERE SerialNum='ABC-123-456'
```

### Query Indexes

```
IX_ProductCode (ProductCode)
  - Find all units of Product 1001

IX_GoalID (GoalID)
  - Count units toward goal

IX_PrintDate (PrintDate)
  - Find units made on specific date

IX_KillDate (KillDate)
  - Find units expiring soon (FIFO)

IX_Sent (Sent)
  - Find units not yet shipped

IX_Shift (Shift)
  - Analyze by shift

IX_PalletNum (PalletNum)
  - Find all units on specific pallet
```

---

## Business Rules for Serial

### Creation Rules

| Rule                     | Details                                             |
| ------------------------ | --------------------------------------------------- |
| **Unique SerialNum**     | No two "A" records with same SerialNum              |
| **Auto-Generation**      | SerialNum must be unique and often system-generated |
| **ProductCode Required** | Every Serial must have valid ProductCode FK         |
| **PrintDate Set**        | Set to TODAY at creation time                       |
| **PrintTime Set**        | Set to SECONDS(MTIME) for per-second precision      |
| **Default Weights**      | Initialized from Product record                     |
| **Operator Captured**    | Must capture current operator for audit trail       |
| **Shift Captured**       | Must capture current shift (1-3)                    |
| **ScaleNumber Captured** | Must capture which scale produced it                |

### Weight Rules

| Rule                  | Details                                        |
| --------------------- | ---------------------------------------------- | ----------------- | --------------------------- |
| **LabelWgt ≤ MaxWgt** | Cannot exceed product maximum weight           |
| **NetWgt ≥ MinWgt**   | Must meet product minimum weight               |
| **Variance Check**    | IF                                             | NetWgt - Expected | > Tolerance: Warn operators |
| **Tare Calculation**  | TotalTareWgt = BoxTare + ExtraTare + ItemTares |
| **Weight Units**      | Consistent per product (LB or KG)              |

### Date Rules

| Rule                                     | Details                                     |
| ---------------------------------------- | ------------------------------------------- |
| **PrintDate ≥ Today**                    | Can't print in the past                     |
| **KillDate = PackDate + ProductDays**    | Expiration based on product shelf life      |
| **SellByDate = KillDate + SellByOffset** | Calculated at label print time              |
| **PackDate Accuracy**                    | Should match actual production time         |
| **Offsets Applied**                      | SellByOffset, VarOffset, etc., modify dates |

### State Rules

| Rule                         | Details                                 |
| ---------------------------- | --------------------------------------- |
| **Can't Create Duplicate A** | SerialNum can only be created once      |
| **Can Only Delete Once**     | Only one D record per A record          |
| **Sent Flags**               | Once Sent=YES, may be archived/readonly |
| **Reprinted Tracking**       | Track if label was reprinted            |
| **Goal Link Optional**       | GoalID can be 0 (off-goal production)   |

### Data Integrity Rules

| Rule                   | Details                                             |
| ---------------------- | --------------------------------------------------- |
| **ProductCode exists** | FK check to Product table                           |
| **Operator exists**    | FK check to Operator table                          |
| **ItemCode valid**     | If specified, must exist in Item table              |
| **Lot Alphanumeric**   | Max 10 chars, alphanumeric                          |
| **SerialNum Format**   | System-defined, usually alphanumeric up to 12 chars |

---

## Real-World Scenarios

### Scenario 1: Simple Retail Bottle Production

```
User Action:
  1. Operator scans product "102" (2L Milk)
  2. Place bottle on scale
  3. Weight = 2.05 lbs
  4. System prints barcode label

Serial Created:
  SerialNum:      "20260311-0001"
  AddOrDel:       "A"
  ProductCode:    102
  PrintDate:      2026-03-11
  PrintTime:      1430567
  PackDate:       2026-03-11
  Operator:       "jsmith"
  Shift:          2
  ScaleNumber:    1
  LabelWgt:       2.05
  NetWgt:         1.98
  BoxTare:        0.07
  KillDate:       2026-03-18 (7-day shelf life)
  SellByOffset:   3
  SellByDate:     2026-03-21 (KillDate + 3 days)
  GoalID:         5432 (production goal ID)
  Sent:           NO
```

### Scenario 2: Multi-Item Case with Individual Tracking

```
Product: 6-pack of items
Each item must be individually tracked

Serial Created (Case):
  SerialNum:      "MP-20260311-00156"
  ProductCode:    3100
  ItemCode:       (None, case-level)
  LabelWgt:       4.50 lbs (total case)
  NetWgt:         4.20 lbs (product only)

ItemSerial Records Created (6 items):
  Each with:
    CaseSerialNum:  "MP-20260311-00156"
    ProductCode:    3100
    ItemCode:       5021 (item code)
    NetWgt:         0.70 lbs (per item)
    ItemPreTareWgt: 0.05 lbs
    Price:          1.49 (per item)

  Total: 6 items × 0.70 = 4.20 lbs ✓
```

### Scenario 3: Weight Variance and Rejection

```
User Action:
  1. Product is 1 lb steaks (±0.05 oz tolerance)
  2. Scale reads 1.12 lbs (exceeds max)

System Response:
  Serial created but with warning

Serial Record:
  SerialNum:      "STK-20260311-0789"
  ProductCode:    2050
  LabelWgt:       1.12 lbs
  NetWgt:         1.12 lbs
  Status Message: "Weight 1.12 exceeds max 1.10"

Operator Options:
  - Accept and print anyway
  - Return to scale and reweigh
  - Reject the unit (delete)
```

### Scenario 4: Cancellation/Return

```
Problem: Unit was defective, needs removal

Original Serial (A record):
  SerialNum:      "ABC-123-456"
  AddOrDel:       "A"
  ProductCode:    1001
  GoalID:         5432
  (All normal data)

Cancellation Serial (D record):
  SerialNum:      "ABC-123-456"
  AddOrDel:       "D"
  ProductCode:    1001
  CreationDate:   (next day, when defect discovered)

Goal Adjustment:
  Original Goal Count: 1000 + 1 serial = 1001
  After Cancellation: 1001 - 1 = 1000
  (Unit no longer counts toward goal)
```

### Scenario 5: Pallet Shipment

```
Multiple Serials packed into pallet

PalletHdr Created:
  PalletNum:      "99YDDD0001"
  CaseCount:      50
  Complete:       YES
  DateComplete:   2026-03-11
  Sent:           NO

PalletDtl Records (50 entries):
  Each references one Serial:
    PalletNum:    "99YDDD0001"
    SerialNum:    "ABC-123-001", LabelWgt: 2.5
    SerialNum:    "ABC-123-002", LabelWgt: 2.5
    SerialNum:    "ABC-123-003", LabelWgt: 2.5
    ...
    SerialNum:    "ABC-123-050", LabelWgt: 2.5
    Total: 125 lbs

Then:
  All Serials updated:
    Serial.PalletNum = "99YDDD0001"
    Serial.Sent = YES

  PalletHdr.Sent = YES
```

---

## Common Queries

### Find All Production Today

```sql
SELECT
  SerialNum,
  ProductCode,
  NetWgt,
  LabelWgt,
  Operator,
  PrintDate
FROM Serial
WHERE PrintDate = TODAY
  AND AddOrDel = 'A'
ORDER BY SerialNum;
```

### Count Units Toward Goal

```sql
SELECT
  COUNT(*) as UnitCount,
  SUM(LabelWgt) as TotalWeight
FROM Serial
WHERE GoalID = 5432
  AND AddOrDel = 'A';

-- Result: 892 units, 2230.5 lbs
```

### Find Expired/Soon-to-Expire

```sql
SELECT
  SerialNum,
  KillDate,
  DAYS-BETWEEN(TODAY, KillDate) as DaysUntilExpire
FROM Serial
WHERE KillDate BETWEEN TODAY AND TODAY + 7
  AND AddOrDel = 'A'
  AND Sent = NO
ORDER BY KillDate;
```

### Production by Operator

```sql
SELECT
  Operator,
  COUNT(*) as UnitsProduced,
  SUM(LabelWgt) as TotalWeight,
  AVG(LabelWgt) as AvgWeight
FROM Serial
WHERE PrintDate = TODAY
  AND AddOrDel = 'A'
GROUP BY Operator
ORDER BY UnitsProduced DESC;
```

### Find Serials on Specific Pallet

```sql
SELECT
  s.SerialNum,
  s.ProductCode,
  s.NetWgt,
  p.LabelWgt as PalletLabelWgt
FROM Serial s
JOIN PalletDtl p ON s.SerialNum = p.SerialNum
WHERE p.PalletNum = '99YDDD0001'
ORDER BY s.SerialNum;
```

### Variance Analysis

```sql
SELECT
  SerialNum,
  LabelWgt,
  NetWgt,
  (LabelWgt - NetWgt) as Variance,
  CASE
    WHEN ABS(LabelWgt - NetWgt) > 0.1 THEN 'ERROR'
    WHEN ABS(LabelWgt - NetWgt) > 0.05 THEN 'WARNING'
    ELSE 'OK'
  END as Status
FROM Serial
WHERE ProductCode = 1001
  AND PrintDate = TODAY
  AND AddOrDel = 'A'
ORDER BY ABS(Variance) DESC;
```

---

## Weight Calculation Examples

### Example 1: Simple Case

```
Product: Steak Package
  ProductCode = 2001
  MinWgt = 0.90
  MaxWgt = 1.10
  BoxTare (fixed) = 0.15 lbs

Production:
  Scale reads = 1.05 lbs
  NetWgt = 1.05 - 0.15 = 0.90 lbs
  LabelWgt = 1.05 lbs

Validation:
  MinWgt ≤ NetWgt ≤ MaxWgt
  0.90 ≤ 0.90 ≤ 1.10 ✓ OK
```

### Example 2: Multi-Tare Case

```
Product: Frozen dinner in tray with film
  ProductCode = 1500

Weights:
  Tray (pre-fill) = 0.30 lbs
  Food = 1.20 lbs
  Film/seal = 0.05 lbs
  Ice layer = 0.10 lbs (moisture)

Serial Record:
  ContainerPreTareWgt = 0.30 (tray)
  ItemPostTareWgt = 0.05 (film on item)
  ExtraTare = 0.10 (ice)
  TotalTareWgt = 0.30 + 0.05 + 0.10 = 0.45 lbs

  NetWgt = 1.20 lbs (actual food)
  LabelWgt = 1.20 + 0.45 = 1.65 lbs

Validation:
  NetWgt + TotalTareWgt = LabelWgt
  1.20 + 0.45 = 1.65 ✓
```

### Example 3: Variance Detection

```
Product: 2 lb roast
  Expected: 2.00 lbs ± 0.05 tolerance
  Acceptable Range: [1.95, 2.05]

Production Run:
  Case 1: 1.98 lbs ✓ (within range)
  Case 2: 2.04 lbs ✓ (within range)
  Case 3: 2.12 lbs ✗ (exceeds max 2.05)
  Case 4: 1.92 lbs ✗ (below min 1.95)
  Case 5: 2.01 lbs ✓ (within range)

System Warnings:
  Case 3: "Weight 2.12 lbs exceeds maximum 2.05"
  Case 4: "Weight 1.92 lbs below minimum 1.95"
```

---

## Data Validation Checklist

When creating/updating Serial, system validates:

- [ ] SerialNum is unique within (SerialNum, AddOrDel)
- [ ] ProductCode exists in Product table
- [ ] If specified, Operator exists in Operator table
- [ ] If specified, ItemCode exists in Item table
- [ ] LabelWgt within Product.MinWgt to Product.MaxWgt
- [ ] NetWgt ≥ 0
- [ ] All tare weights ≥ 0
- [ ] PrintDate ≤ TODAY
- [ ] PackDate ≤ TODAY
- [ ] KillDate ≥ PackDate
- [ ] SellByDate calculated correctly from KillDate + SellByOffset
- [ ] If GoalID ≠ 0, GoalID exists in Goals table
- [ ] If PalletNum specified, exists in PalletHdr
- [ ] WgtUnits in ("LB", "KG")
- [ ] Lot is alphanumeric, ≤ 10 chars
- [ ] Shift in (1, 2, 3)
- [ ] ScaleNumber in valid range

---

## Performance Considerations

### Large-Volume Production

For high-speed production (thousands of serials/hour):

1. **Bulk Inserts** - Insert multiple serials in transaction
2. **Index Strategy** - Use composite PK + selective secondary indexes
3. **Archive Old Data** - Move historical serials to archive table
4. **Statistics** - Use Totals table for aggregate queries instead of SUM(Serial)

### Query Optimization

```
SLOW: SELECT SUM(LabelWgt) FROM Serial WHERE PrintDate = TODAY
      (full table scan if no index)

FAST: SELECT TotalLabelWgt FROM Totals
      WHERE PdnDate = TODAY
      (pre-aggregated, instant lookup)
```

### Archival Strategy

Keep recent serials in active table (~3 months), archive older:

- Active table: Fast queries, writes
- Archive table: Historical queries, reporting
- Summary table (Totals): Real-time metrics

---

## Integration Points

### ERP/Inventory System

When `Serial.Sent = NO`:

1. System aggregates serials by order
2. Exports to ERP (SAP, NetSuite, etc.)
3. Updates inventory
4. Sets `Serial.Sent = YES`

### Quality Management System (QMS)

Links serials to:

- Lot numbers
- Expiration dates
- Production dates
- Shift/operator audit trail
- Allows rapid identification for recalls

### Traceability Systems

FSMA, FSSC 22000, SQF compliance:

- One-step forward: Find all serials by customer
- One-step back: Find source batch/lot
- Internal traceability: Operator, shift, scale, time

---

## Summary

| Aspect             | Details                                                  |
| ------------------ | -------------------------------------------------------- |
| **Purpose**        | Master production record for traceability                |
| **Scope**          | One per individual case/unit produced                    |
| **Primary Key**    | SerialNum (composite with AddOrDel)                      |
| **Volume**         | Thousands to millions per facility per day               |
| **Lifecycle**      | Create → Weigh → Pack → Ship → Archive                   |
| **Key Fields**     | SerialNum, ProductCode, PrintDate, KillDate, Weights     |
| **Critical Links** | Goals, Product, Operator, PalletHdr                      |
| **Compliance**     | Food safety traceability, regulatory proof               |
| **Queries**        | Goal measurement, variance analysis, expiration tracking |
| **Archive**        | 90+ days to historical storage                           |

---

## References

- **Table:** `Serial` in legacy-database-schema.md
- **Related:** `Product`, `Goals`, `Totals`, `PalletHdr/PalletDtl`, `ItemSerial`, `Operator`
- **UI:** Browsed/viewed via v-serials.w, v-serial-main.w
- **Creation:** During ProductProcess execution (case-print.p, weigh.p)
- **Maintenance:** Serial maintenance screens, deletion/cancellation screens
- **Reporting:** Serial summary reports, variance reports, compliance reports
