# ProductProcess: Complete Concept Guide

## Overview

**ProductProcess** is the core manufacturing configuration mechanism in the SLC application that defines **how a specific product is manufactured** by sequences of steps and processes.

It's a **bridge/junction table** that:
- Maps each **Product** to one or more **ValidProcess** entries
- Defines the **sequence and order** of manufacturing steps
- Specifies **optional hardware devices** required at each step
- Enables **flexible manufacturing workflows** without code changes

**Think of it as:** A recipe or bill-of-operations (BOM) for manufacturing operations where each step is a distinct process.

---

## Why ProductProcess Exists

### Problem It Solves

Different products require different manufacturing sequences:
- Product A: `Print Case Label → Scale/Weigh → Print Item Labels`
- Product B: `Print Case Label → Print Item Labels → Scale/Weigh → Variable Labeling`
- Product C: `Scale/Weigh → Print Case Label → Pallet Output`

Without ProductProcess, you'd need:
- Hardcoded if-statements for each product
- Code changes to modify a workflow
- Maintenance burden

**ProductProcess solves this** by making manufacturing workflows configurable in the database.

### Business Benefits

✅ **Flexibility** - Different products have different manufacturing flows  
✅ **No Code Changes** - Update workflows via database maintenance screens  
✅ **Runtime Selection** - Choose which process sequence to use  
✅ **Device Management** - Specify which hardware devices are needed  
✅ **Sequence Control** - Enforce strict order of operations  
✅ **Extensibility** - Add new processes without touching existing configurations  

---

## Table Schema

### ProductProcess Table

**Primary Key:** `ProductCode` + `ProcessSequence`

```sql
CREATE TABLE ProductProcess (
    ProductCode        INT NOT NULL,        -- PK1: Which product
    ProcessSequence    INT NOT NULL,        -- PK2: What order (1, 2, 3...)
    ProcessID          CHAR(20),            -- FK: Reference to ValidProcess
    GroupID            CHAR(20),            -- Logical grouping
    ButtonLabel        CHAR(20),            -- UI button label
    ButtonImage        CHAR(10),            -- Icon filename (.bmp)
    InputProcess       LOGICAL DEFAULT NO,  -- Is this an input step?
    Device             CHAR(15),            -- Optional device (FK to Device)
    AlternateDevice    CHAR(15),            -- Fallback device if primary unavailable
    Repetitions        INT DEFAULT 0,       -- How many times to repeat
    ResetRequired      LOGICAL DEFAULT NO,  -- Must scale be zeroed?
    ResetWgt           DECIMAL DEFAULT 0,   -- Expected weight threshold
    Routine            CHAR(15),            -- Input capture or output formatting routine
    SetupRoutine       CHAR(15),            -- Setup/initialization routine
    OutputLabel        CHAR(15),            -- Label file for output
    EscapeButtonLabel  CHAR(15),            -- Exit button text
    Description        CHAR(20),            -- Human-readable step description
    Item               INT DEFAULT 0,       -- Optional item code
    AccumForPrior      LOGICAL DEFAULT NO,  -- Sum tare+weight for prior steps
    WgtType            CHAR(8)              -- "Tare", "Case", "Item" - what weight type
);

-- Primary Key Index
CREATE UNIQUE INDEX PK_ProductProcess ON ProductProcess(ProductCode, ProcessSequence);
```

### ValidProcess Table (Reference)

```sql
CREATE TABLE ValidProcess (
    ProcessID          CHAR(20) NOT NULL PRIMARY KEY,
    Description        CHAR(40),            -- What this process does
    DefaultProcess     CHAR(15)             -- Default variant
);
```

---

## Understanding ProductProcess Fields

### Core Fields

| Field | Purpose | Example |
|-------|---------|---------|
| **ProductCode** | Which product uses this sequence | 12345 |
| **ProcessSequence** | Order of execution (1=first, 2=second) | 1, 2, 3, 4 |
| **ProcessID** | Which process type (FK to ValidProcess) | "CaseWeight", "ItemPrint", "PalletOutput" |
| **GroupID** | Logical grouping of related steps | "Input", "Output", "Reports" |

### UI/Display Fields

| Field | Purpose | Example |
|-------|---------|---------|
| **ButtonLabel** | Text shown on UI button | "Weigh Case" |
| **ButtonImage** | Image file for button icon | "scale.bmp" |
| **Description** | Human description | "Weigh each case" |
| **EscapeButtonLabel** | Exit/Cancel button text | "Skip Weight" |

### Hardware Fields

| Field | Purpose | Example |
|-------|---------|---------|
| **Device** | Primary hardware device required | "Scale1" (references Device.DeviceID) |
| **AlternateDevice** | Fallback device if primary unavailable | "Scale2" |

### Execution Control Fields

| Field | Purpose | Example |
|-------|---------|---------|
| **InputProcess** | Is this a data input step? | "Y" = get operator input |
| **Routine** | Code routine to execute | "scale-input", "label-output" |
| **SetupRoutine** | Initialization before main routine | "scale-calibrate" |
| **Repetitions** | How many times to repeat this step | 3 (scale weigh might repeat 3x) |
| **ResetRequired** | Must scale be zeroed before use? | YES |
| **ResetWgt** | Expected weight for scale validation | 5.0 |

### Weight/Data Fields

| Field | Purpose | Example |
|-------|---------|---------|
| **WgtType** | What weight type is captured | "Tare", "Case", "Item" |
| **AccumForPrior** | Accumulate weight from prior steps? | YES (for combined weights) |
| **Item** | If process uses item codes | 5001 |
| **OutputLabel** | Label format file | "Case.lbl", "Item.lbl" |

---

## Common ProcessID Types

These are standard process types used throughout the system:

### Input/Capture Processes

| ProcessID | Purpose |
|-----------|---------|
| **CaseWeight** | Scale/weigh a case using scale device |
| **ItemWeight** | Weigh individual items |
| **ContainerPreTare** | Capture pre-packaging tare weight |
| **Container-PreTare** | Alternative naming variant |
| **KeyboardInput** | Manual keyboard data entry |
| **BarcodeInput** | Scan barcode for data capture |

### Output/Formatting Processes

| ProcessID | Purpose |
|-----------|---------|
| **CasePrint** | Print case/box label |
| **ItemPrint** | Print item label |
| **CaseOutput** | Case-level output formatting |
| **Item Output** | Item-level output formatting |
| **Pallet Output** | Pallet-level output formatting |
| **PrintLabel** | Generic label printing |

### Data Processing Processes

| ProcessID | Purpose |
|-----------|---------|
| **VariableData** | Process variable/dynamic data |
| **DateProcessing** | Handle date codes and kill dates |
| **PriceUpdate** | Update pricing information |

### Quality/Validation Processes

| ProcessID | Purpose |
|-----------|---------|
| **WeightValidation** | Validate weight is within acceptable range |
| **CountValidation** | Validate item count matches target |
| **DataValidation** | Generic data validation |

### System Processes

| ProcessID | Purpose |
|-----------|---------|
| **Group** | Logical group/folder (not an executable step) |
| **Default** | Use system default behavior |
| **DefaultCaseWeight** | Use default case weight (legacy) |

---

## Real-World Example: Product Manufacturing Sequences

### Example 1: Simple Product (Fluid/Retail)

**Product Code:** 1001  
**Description:** 2L Milk Bottle

```
ProcessSequence  ProcessID              Device          Description
─────────────────────────────────────────────────────────────────
     1          CasePrint              Printer1        Print case label
     2          CaseWeight             Scale1          Weigh case
     3          ItemPrint              Printer2        Print item labels
     4          PalletOutput           (none)          Format pallet data
```

**Flow:** 
1. Operator scans/enters product
2. Case label prints
3. Operator places case on Scale1
4. System captures weight
5. Item labels print
6. System generates pallet output

### Example 2: Complex Product (Multi-Item)

**Product Code:** 2050  
**Description:** 3-pack with multiple items

```
ProcessSequence  ProcessID              Device          Description
─────────────────────────────────────────────────────────────────
     1          CasePrint              Printer1        Print case label
     2          ItemPrint              Printer3        Print item stickers (3x)
     3          ContainerPreTare       Scale2          Tare container
     4          ItemWeight             Scale2          Weigh item 1
     5          ItemWeight             Scale2          Weigh item 2
     6          ItemWeight             Scale2          Weigh item 3
     7          CaseWeight             Scale2          Final case weight
     8          VariableData           (none)          Process variable fields
     9          PalletOutput           (none)          Generate pallet record
```

**Flow:**
1. Case label prints
2. Item stickers print (3 times)
3. Pre-tare container on scale
4. Weigh individual items (scale reset between each)
5. Weigh final case
6. Process variable data (date, lot, etc.)
7. Generate pallet documentation

### Example 3: Batch/Production Process

**Product Code:** 3100  
**Description:** Bulk batch production

```
ProcessSequence  ProcessID              Device          Description
─────────────────────────────────────────────────────────────────
     1          KeyboardInput          (none)          Enter batch info
     2          BarcodeInput           Scanner1        Scan component barcodes
     3          ContainerPreTare       Scale1          Tare batch container
     4          CaseWeight             Scale1          Weigh batch (with resets)
     5          CasePrint              Printer1        Print batch label
     6          VariableData           (none)          Batch metadata
```

**Flow:**
1. Operator enters batch date, lot number
2. Scan component barcodes for traceability
3. Setup batch container tare
4. Weigh batch multiple times with resets
5. Print completed batch label
6. Store batch metadata

---

## Relationships to Other Concepts

### 1. Product ← ProductProcess → ValidProcess (Many-to-Many)

```
Product (1:N) ──→ ProductProcess ←(N:1)─── ValidProcess

One Product can use MANY Processes
One ValidProcess can be used by MANY Products
```

**Example:**
- Product 1001 uses: [CasePrint → CaseWeight → ItemPrint]
- Product 1002 uses: [CasePrint → ItemPrint → CaseWeight]
- Product 1003 uses: [CaseWeight → CasePrint]
- ProcessID "CasePrint" is used by all three products

### 2. ProductProcess ← Device (Optional FK)

```
ProductProcess (N:1) ──→ Device

Each ProcessStep can optionally require a Device
Device contains: ComPort, IPAddress, TimeoutMS, Model, etc.
```

**Example:**
```
ProcessSequence 1: ProcessID="CaseWeight", Device="Scale1"
  → Look up Device where DeviceID="Scale1"
  → Get IPAddress="192.168.1.50", Baud=9600, TimeoutMS=5000
  → Use these settings to communicate with actual scale hardware
```

### 3. ProductProcess → Sequences (Ordering)

```
Product → ProductProcess (ordered by ProcessSequence)
           ├── Step 1: CasePrint
           ├── Step 2: CaseWeight
           ├── Step 3: ItemPrint
           └── Step 4: PalletOutput
```

The **ProcessSequence field enforces strict ordering**:
- Steps are executed in ascending ProcessSequence order
- ProcessSequence must have no gaps (1, 2, 3, 4 - not 1, 2, 5, 7)
- Missing or out-of-order sequences cause validation errors

### 4. ProductProcess → Routines (Code Execution)

```
Routine field:
  "scale-input"    → execute scale-input.p
  "label-output"   → execute label-output.p
  "keyboard-entry" → execute keyboard-entry.p
```

Each ProductProcess can call custom code routines:
- **Routine** - Main executable code
- **SetupRoutine** - Pre-processing/initialization
- **OutputLabel** - Label format to use

### 5. ProductProcess → Item (Optional Item Link)

```
ProductProcess.Item (FK to Item.ItemCode)
  → Used when a process needs to reference specific item configurations
  → Example: ItemPrint process uses Item.LabelFile from Item table
```

---

## How ProductProcess Works in Practice

### Scenario: Product Starting with "Create Serial"

**Step 1: Product Selected**
```
User scans barcode or enters ProductCode 1001
System loads Product record
```

**Step 2: Load ProcessSequence**
```
SELECT * FROM ProductProcess
WHERE ProductCode = 1001
ORDER BY ProcessSequence ASC

Results:
┌─────────────────────────────────────┐
│ ProcessSequence│ProcessID │Device   │
├─────────────────────────────────────┤
│       1        │CasePrint │Printer1 │
│       2        │CaseWeight│Scale1   │
│       3        │ItemPrint │Printer2 │
│       4        │PalletOut │(none)   │
└─────────────────────────────────────┘
```

**Step 3: Execute Sequence**
```
FOR EACH ProductProcess (sequence 1-4):
  
  STEP 1 - ProcessID="CasePrint"
    ├─ Load Device Printer1 settings
    ├─ Call Routine (if defined)
    ├─ Generate Label from OutputLabel
    ├─ Send to printer

  STEP 2 - ProcessID="CaseWeight"  
    ├─ Load Device Scale1 settings (IP, port, timeout)
    ├─ Call SetupRoutine (reset scale)
    ├─ Call Routine (capture weight)
    ├─ Validate ResetRequired, Repetitions
    ├─ Store in Serial.NetWgt

  STEP 3 - ProcessID="ItemPrint"
    ├─ Load Device Printer2 settings
    ├─ Call Routine
    ├─ Generate items based on Desc1/Desc2
    ├─ Send to printer

  STEP 4 - ProcessID="PalletOutput"
    ├─ Call Routine (format pallet data)
    ├─ Update pallet tracking tables
    ├─ Log completion
```

---

## ProcessSequence: Ordering & Gaps

### Valid Sequences

```
✓ ProcessSequence: 1, 2, 3, 4          (continuous, no gaps)
✓ ProcessSequence: 10, 20, 30          (gaps OK, checks order)
✓ ProcessSequence: 1 (single step)     (allowed)
```

### Invalid Sequences

```
✗ ProductCode=1001, Seq=1 MISSING
  (if Seq 2,3,4 exist without 1)

✗ ProductCode=1001, Seq out of order in execution
  (if not sorted/ordered properly)
```

### Validation Logic

When system needs to execute ProductProcess:
```progress
DEFINE VARIABLE v-LastSeq AS INT.
v-LastSeq = 0.

FOR EACH ProductProcess WHERE ProductCode = cProductCode
                          ORDER BY ProcessSequence:
    
    IF ProcessSequence <= v-LastSeq THEN DO:
        MESSAGE "Sequence out of order" VIEW-AS ALERT-BOX ERROR.
        RETURN.
    END.
    
    v-LastSeq = ProcessSequence.
    
    /* Execute this step */
    RUN execute-process(ProcessID, Device, Routine).
END.
```

---

## GroupID: Logical Organization

**GroupID** groups related ProcessSequence steps together:

```
ProductCode=1001, ProcessSequence Order:

GroupID          ProcessID           Purpose
─────────────────────────────────────────
"Input"    ──┐   KeyboardInput     ├─ Data capture
             └─  BarcodeInput      │

"Processing" ─  CaseWeight          ├─ Core operations  
             ─  VariableData       │

"Output"    ──┐  CasePrint         ├─ Finalization
             ├─ ItemPrint         │
             └─ PalletOutput      │
```

**Use:** UI organization, logical process grouping, conditional execution

---

## Input vs. Output Processes

### InputProcess = YES (Data Capture)

```
ProcessID      WgtType           Purpose
─────────────────────────────────────────
CaseWeight     "Case"            Operator inputs case weight
ItemWeight     "Item"            Operator inputs item weight
KeyboardInput  (varies)          Keyboard data entry
BarcodeInput   (varies)          Barcode scanning
```

Characteristics:
- Waits for **operator input**
- Captures **weight/data** from external source
- Updates **Serial** or **Modify** table
- May show **UI dialogs** for confirmation

### InputProcess = NO (Processing/Output)

```
ProcessID      Purpose
──────────────────────
CasePrint      Label printing
ItemPrint      Item label generation
PalletOutput   Pallet data formatting
VariableData   Process dynamic fields
```

Characteristics:
- **Automatic** execution (no wait)
- Reads from **Serial** table
- Generates **output** (labels, data, reports)
- No operator interaction needed

---

## Device Integration

### Device Fields

Each ProductProcess can specify:

```
Device                    ← Primary device (REQUIRED if step needs hardware)
AlternateDevice           ← Fallback device (optional)
```

### Device Lookup

```progress
/* Find the device for this step */
FIND Device WHERE Device.DeviceID = ProductProcess.Device
    NO-LOCK NO-ERROR.

IF NOT AVAIL(Device) THEN DO:
    /* Try alternate device */
    FIND Device WHERE Device.DeviceID = ProductProcess.AlternateDevice
        NO-LOCK NO-ERROR.
        
    IF NOT AVAIL(Device) THEN DO:
        MESSAGE "No device available for: " + ProcessID
            VIEW-AS ALERT-BOX ERROR.
        RETURN ERROR.
    END.
END.

/* Initialize device with settings */
Device.IPAddress    → "192.168.1.50"
Device.Baud         → 9600
Device.ComPort      → 1
Device.TimeoutMS    → 5000
Device.Model        → "Mettler Toledo IND690"
```

### Real Example: Scale Device

```
ProductProcess:
  ProcessID = "CaseWeight"
  Device = "Scale1"
  ResetRequired = YES

Device Record (DeviceID="Scale1"):
  Model = "Mettler Toledo IND690"
  IPAddress = "192.168.1.50"
  Baud = 9600
  TimeoutMS = 5000
  ComPort = 1
  
System:
  1. Opens communication to 192.168.1.50:9600
  2. Sends RESET command (ResetRequired=YES)
  3. Waits up to 5 seconds for response
  4. Reads weight value
  5. Validates weight >= 0 and <= MaxWgt
  6. Returns weight or error
```

---

## Weight Type: Tare, Case, Item

The **WgtType** field indicates what type of weight is being captured:

### WgtType = "Tare"
```
Weight of packaging/container WITHOUT product
Example: "Plastic tray weighs 0.5 lbs"
Storage: Serial.ContainerPreTareWgt
Uses: Calculate net weight by subtracting from total
```

### WgtType = "Case"  
```
Weight of complete case/box (with products)
Example: "Case weighs 10.5 lbs total"
Storage: Serial.LabelWgt or Serial.NetWgt
Uses: Label printing, pallet stacking, goal achievement
```

### WgtType = "Item"
```
Weight of individual item WITHOUT packaging
Example: "Steak weighs 0.75 lbs"
Storage: Serial.NetWgt or ItemSerial.NetWgt
Uses: Item-level pricing, variance calculations
```

---

## Reset and Validation: ResetRequired & ResetWgt

### ResetRequired = YES

```
Step: CaseWeight
ResetRequired = YES

Execution:
  1. Display "Place case on scale"
  2. Send RESET command to Scale device
  3. Send TARE command to set baseline to 0
  4. Display "Ready to weigh"
  5. Capture weight reading
```

### ResetWgt

Expected weight for validation:

```
Step 1: CaseWeight
  ResetRequired = YES
  ResetWgt = 5.0
  
Execution:
  1. Send RESET to scale
  2. Verify scale returns to ≈ 5.0 lbs baseline
  3. If significantly different, show warning:
     "Scale reset value 4.8 lbs != expected 5.0 lbs"
```

---

## Repetitions: Multiple Captures

**Repetitions** field allows capturing the same measurement multiple times:

```
ProcessSequence   ProcessID        Repetitions    Description
────────────────────────────────────────────────
     1          CaseWeight           3          "Weigh case 3 times"
     2          ItemWeight           5          "Weigh each item (5x)"
```

**Scenario: Multi-Item Weigh**

```
Product has 5 items in a case
ProcessID = "ItemWeight", Repetitions = 5

Execution:
  FOR i = 1 TO 5:
    Display "Weigh item " + i
    Read weight from scale
    Reset scale
    Capture weight #i
  END.
  
  Calculate: Total weight = Item1 + Item2 + Item3 + Item4 + Item5
```

---

## Accumulation: AccumForPrior

**AccumForPrior** = YES accumulates weight from all prior steps:

### Example: Multi-Component Case

```
ProcessSequence  ProcessID        WgtType    AccumForPrior  
────────────────────────────────────────────────────
    1           ContainerTare     Tare       NO
    2           Item1Weight       Item       NO
    3           Item2Weight       Item       NO
    4           CaseWeight        Case       YES ← Accumulate prior
```

**Execution:**
```
Step 1: Tare = 0.5 lbs (stored separately)
Step 2: Item1 = 1.2 lbs (in Modify)
Step 3: Item2 = 0.8 lbs (in Modify)
Step 4: CaseWeight (AccumForPrior=YES)
  ├─ Read actual weight from scale: 2.8 lbs
  ├─ Accumulate from prior: 0.5 + 1.2 + 0.8 = 2.5 lbs
  ├─ Compare: Actual = 2.8, Expected = 2.5
  ├─ If difference > tolerance: Show warning
  └─ Store final: Serial.NetWgt = 2.8 lbs (actual)
```

---

## ProcessingMode: Runtime Selection

Some products can be manufactured using **different process sequences**.

**ProcessingMode** (stored in Modify table) allows operators to **select which ProductProcess sequence to use**:

```
Product 1001 has TWO possible sequences:

Sequence A (ProcessingMode="FastTrack"):
  1. CasePrint
  2. CaseWeight
  3. PalletOutput
  
Sequence B (ProcessingMode="FullQuality"):
  4. CasePrint
  5. CaseWeight
  6. ItemPrint
  7. DataValidation
  8. PalletOutput

At runtime:
  Operator selects ProcessingMode = "FullQuality"
  System loads ProcessSequence for mode "FullQuality"
  All steps executed in order
```

**Use:** Support different production strategies for same product

---

## Data Types and Constraints

### Field Types

| Field | Type | Constraint |
|-------|------|-----------|
| ProductCode | INT | PK, FK to Product |
| ProcessSequence | INT | PK, must be positive |
| ProcessID | CHAR(20) | FK to ValidProcess |
| GroupID | CHAR(20) | (optional) |
| ButtonLabel | CHAR(20) | UI display |
| Device | CHAR(15) | FK to Device (optional) |
| Repetitions | INT | >= 0 |
| ResetWgt | DECIMAL | >= 0 |
| WgtType | CHAR(8) | "Tare", "Case", "Item" |

### Uniqueness

- **Primary Key:** `(ProductCode, ProcessSequence)` - Each product process must be unique per sequence
- **No Duplicate Sequences:** Can't have ProductCode=1001, Seq=1 twice
- **ProcessID Validation:** Must exist in ValidProcess table

---

## Validation Rules

### Cross-Table Validation (ProductProcess-RI.p)

```progress
/* For each ProductProcess record: */

1. ProcessID must exist in ValidProcess table
   ── If missing: Error "Invalid ProcessID"
   
2. Device (if specified) must exist in Device table
   ── If missing: Error "Device not found"
   
3. AlternateDevice (if specified) must exist in Device table
   ── If missing: Warning or skip
   
4. If InputProcess=YES, Routine must be defined
   ── If missing: Error "Routine required for input"
   
5. ProcessSequence must be unique per ProductCode
   ── If duplicate: Error "Duplicate ProcessSequence"
```

---

## Related Concepts

### 1. **Goals** → ProductProcess

When a **Goal** is created for a product:
- System loads ProductProcess for that ProductCode
- Uses ProcessSequence to determine manufacturing steps
- Assigns goals metrics (count target, weight target) to the process flow

### 2. **Serial** → ProductProcess

When a **Serial** is created:
- ProductProcess determines which processes to execute
- Each step captures data into Serial table:
  - Serial.PrintDate
  - Serial.PackDate
  - Serial.KillDate
  - Serial.LabelWgt
  - Serial.NetWgt
  - Serial.UserBarString (if custom barcode needed)

### 3. **Modify** → ProductProcess

Modify table stores **runtime configuration per product**:
- ModText1, ModText2 - Override process-level settings
- Price - Used by output processes
- PackDateOffset - Adjust dates in process flow
- PalletNum - Generated during PalletOutput process

### 4. **ItemSerial** → ProductProcess

ItemSerial created when **Item-level process steps** execute:
- ProcessID = "ItemPrint" → ItemSerial records created
- ProcessID = "ItemWeight" → ItemSerial.NetWgt captured
- Each item gets its own SerialNum

### 5. **Device** → ProductProcess

Devices (scales, printers, scanners) are assigned to process steps:
- ProcessSequence 1: Device="Printer1" (print)
- ProcessSequence 2: Device="Scale1" (weigh)
- ProcessSequence 3: Device="Printer2" (print items)

---

## Common Patterns and Use Cases

### Pattern 1: Simple Linear Flow

```
Product: Retail Bottle
Process: Print → Weigh → Output

ProductProcess:
  Seq1: ProcessID="CasePrint", Device="Printer1"
  Seq2: ProcessID="CaseWeight", Device="Scale1"
  Seq3: ProcessID="PalletOutput", Device=(none)
```

### Pattern 2: Multi-Item Branching

```
Product: 6-pack with 6 items
Process: Print Case → Weigh Each Item → Weigh Case → Print Items

ProductProcess:
  Seq1: ProcessID="CasePrint"
  Seq2: ProcessID="ItemWeight", Repetitions=6
  Seq3: ProcessID="CaseWeight"
  Seq4: ProcessID="ItemPrint", Repetitions=6
```

### Pattern 3: Optional Device Fallback

```
Product: Flexible
Process: Weigh with scale (or manual input if scale unavailable)

ProductProcess:
  Seq1: ProcessID="CaseWeight"
         Device="Scale1"
         AlternateDevice="Scale2"
```

### Pattern 4: Conditional Execution

```
Product: Variable configuration
Process: Different steps based on ProcessingMode

Via CrossRef:
  Application="ProcessingMode"
  ID=ProductCode
  Descr="FullQuality" → Load full ProductProcess sequence
  Descr="FastTrack" → Load abbreviated sequence
```

### Pattern 5: Data Accumulation

```
Product: Multi-part assembly
Process: Weigh parts individually, then total

ProductProcess:
  Seq1: ProcessID="ItemWeight", WgtType="Item", AccumForPrior=NO
  Seq2: ProcessID="ItemWeight", WgtType="Item", AccumForPrior=NO  
  Seq3: ProcessID="CaseWeight", AccumForPrior=YES ← Sum all prior weights
```

---

## Troubleshooting and Common Issues

### Issue 1: Process Not Executing

```
Symptom: ProductSequence 2 not running after Seq 1
Cause: ProcessID in Seq 2 doesn't exist in ValidProcess
Fix: Check ProcessID spelling, add to ValidProcess if needed
```

### Issue 2: Device Not Found

```
Symptom: "Scale1 device not found" error
Cause: Device record doesn't exist or DeviceID mismatch
Fix: Create Device record or correct DeviceID in ProductProcess
```

### Issue 3: Invalid Sequence Order

```
Symptom: Processes executing out of order
Cause: ProcessSequence gaps or non-sequential numbers
Fix: Ensure ProcessSequence is 1, 2, 3, 4... (no gaps)
```

### Issue 4: Weight Validation Failure

```
Symptom: "Scale reset value invalid" message
Cause: ResetWgt doesn't match actual scale baseline
Fix: Update ResetWgt value or calibrate scale
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Purpose** | Define manufacturing workflows per product |
| **Key Design** | Bridge table: Product ← ProductProcess → ValidProcess |
| **Primary Key** | ProductCode + ProcessSequence |
| **Flexibility** | Different products = different sequences |
| **Ordering** | ProcessSequence enforces step order |
| **Hardware** | Device field links to actual devices |
| **Execution** | FOR EACH ProductProcess in sequence order |
| **Data Capture** | Stores in Serial, ItemSerial, Modify tables |
| **Validation** | Cross-table referential integrity |
| **UI Integration** | ButtonLabel, ButtonImage for operator interface |
| **Advanced** | Repetitions, AccumForPrior, AlternateDevice |

---

## Complete Data Flow Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                   PRODUCT SELECTION                         │
│                  (User scans barcode)                       │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
         ┌─────────────────────────────┐
         │  Load Product Record         │
         │  ProductCode = 1001          │
         └──────────────┬───────────────┘
                        │
                        ▼
        ┌────────────────────────────────┐
        │ Query ProductProcess WHERE     │
        │ ProductCode = 1001             │
        │ ORDER BY ProcessSequence       │
        └──────────┬─────────────────────┘
                   │
         ┌─────────┴──────────┬──────────────┬─────────────┐
         │                    │              │             │
         ▼                    ▼              ▼             ▼
      Step1             Step2          Step3          Step4
    ProcessID=       ProcessID=     ProcessID=    ProcessID=
    CasePrint      CaseWeight      ItemPrint    PalletOutput
    Device=        Device=         Device=      Device=None
    Printer1       Scale1          Printer2
         │              │              │            │
         ▼              ▼              ▼            ▼
    Print Label   Read Scale    Print Items   Format Pallet
    Printer1      Scale1        Printer2      (no device)
         │              │              │            │
         ▼              ▼              ▼            ▼
    Serial.Descr  Serial.Wgt     ItemSerial   PalletHdr
    Updated       Updated        Created      Updated
         │              │              │            │
         └──────────────┴──────────────┴────────────┘
                        │
                        ▼
              ┌──────────────────────┐
              │  Sequence Complete   │
              │  Case ready to ship  │
              └──────────────────────┘
```

---

## References

- **Table:** `ProductProcess` in legacy-database-schema.md
- **Related:** `Product`, `ValidProcess`, `Device`, `Serial`, `ItemSerial`, `Modify`
- **UI:** Browsed/edited via procurement/configuration screens
- **Validation:** ProductProcess-RI.p enforces referential integrity
- **Utilities:** ProductProcess-recon.p, dbclear.p handle maintenance
