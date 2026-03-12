# Understanding Cross References 

## What Are Cross References?

Cross references in the SLC system are essentially a **flexible key-value lookup mechanism** stored in the `CrossRef` table. Think of it as a universal configuration and mapping storage system that can hold any type of data without requiring database schema changes.

The table has just **3 fields**:

- **Application** - The category or type of data (e.g., "SellByOffset", "ValidMachineIDs")
- **ID** - A specific identifier within that category (e.g., a product code, shift number)
- **Descr** - The actual value or data (e.g., number of days, list of allowed values)

**Key Index:** `(Application, ID)` - This composite unique index is the primary key, meaning each Application/ID pair is unique.

## Why Cross References Instead of Regular Tables?

Cross references solve a significant software engineering problem: **avoiding constant database schema changes**.

### The Problem They Solve:

In a system that needs flexibility for different clients, factories, and configurations:

- ❌ Creating new tables for every configuration type → Schema bloat
- ❌ Adding columns for edge case requirements → Schema creep
- ❌ Database migrations for every new feature → Deployment risk
- ✅ Use CrossRef → One flexible table handles everything

### The Solution:

Instead of creating `SellByOffsetTable`, `ValidMachineIDsTable`, `TrayPrinterConfigTable`, etc., everything goes into one `CrossRef` table with different Application types.

## How They Work: Real-World Examples

### Example 1: Product-Specific Configuration (SellByOffset)

**Scenario:** Different products have different shelf lives (sell-by dates).

**Without CrossRef:**

```sql
CREATE TABLE ProductSellByOffset (
    ProductCode CHAR(5),
    OffsetDays INT
);
```

**With CrossRef:**

```
Application = "SellByOffset"
ID = "12345" (Product Code)
Descr = "30" (Days from pack to sell-by)
Application = "SellByOffset"
ID = "54321" (Product Code)
Descr = "45" (Days from pack to sell-by)
```

**In code:**

```progress
FIND CrossRef WHERE
    CrossRef.Application = "SellByOffset" AND
    CrossRef.ID = "12345"
    NO-ERROR.
IF AVAILABLE(CrossRef) THEN
    vSellByDays = INTEGER(CrossRef.Descr). // Gets 30
```

---

### Example 2: Dynamic Lists (ValidMachineIDs)

**Scenario:** Different facilities can use different machines, and the list changes frequently.

**With CrossRef:**

```
Application = "ValidMachineIDs"
ID = "" (empty)
Descr = "M001,M002,M003,M004,M005"
```

When an operator enters a machine ID, the system validates it against this comma-separated list. To add a new machine, just update that one Descr field—no code changes needed.

**Why this is powerful:**

- Non-technical users (through the CrossRef browser) can add/remove valid machines
- No database schema changes
- No program recompilation
- Real-time effect—takes effect immediately

---

### Example 3: Context-Aware Defaults (LastsForProduct)

**Scenario:** When an operator sets a kill date range for Product A, the next time they load that product, they should see the same kill date range as a default.

**With CrossRef:**

```
Application = "LastsForProduct"
ID = "ProductKillDateRange-12345"
Descr = "03/15/2026" (Last kill date they used)

Application = "LastsForProduct"
ID = "ProductKillDateRange-54321"
Descr = "04/01/2026" (Last kill date for different product)
```

This is used across multiple screens:

- [op-set-killdate-range.w](../print/op-set-killdate-range.w)
- [op-set-dates.w](../print/op-set-dates.w)
- [select-mfgord.w](../print/select-mfgord.w)

**In code:**

```progress
FIND CrossRef WHERE
    CrossRef.Application = "LastsForProduct" AND
    CrossRef.ID = "ProductKillDateRange-" + cProductCode
    NO-ERROR.
IF AVAILABLE(CrossRef) THEN
    cDefaultKillDate = CrossRef.Descr.
```

---

### Example 4: Shift-to-Department Mapping (ZuluDeptsShift)

**Scenario:** When capturing the "Zulu" department field for serials during shift 2, the system needs to know which departments are valid for that shift.

**With CrossRef:**

```
Application = "ZuluDeptsShift"
ID = "1" (Shift 1)
Descr = "Dept1|Dept2|Dept3" (Pipe-separated departments)

Application = "ZuluDeptsShift"
ID = "2" (Shift 2)
Descr = "Dept4|Dept5" (Different departments for different shift)

Application = "ZuluDeptsShift"
ID = "3" (Shift 3)
Descr = "Dept1|Dept6"
```

**In code:**

```progress
FIND CrossRef WHERE
    CrossRef.Application = "ZuluDeptsShift" AND
    CrossRef.ID = STRING(gShift)
    NO-ERROR.
IF AVAILABLE(CrossRef) THEN DO:
    // CrossRef.Descr = "Dept1|Dept2|Dept3"
    // Split and populate dropdown
    sel-ZuluDept:LIST-ITEMS = CrossRef.Descr.
END.
```

---

### Example 5: Dynamic Product Families (Family-{ProductCode})

**Scenario:** Product families come in different weights. Product 12345 (family) might have weights 1.0 lb, 2.0 lb, 3.0 lb.

**With CrossRef:**

```
Application = "Family-12345"
ID = "0001.00" (Weight in lbs)
Descr = "5" (5 units authorized at this weight)

Application = "Family-12345"
ID = "0002.00" (Weight in lbs)
Descr = "10" (10 units authorized at this weight)

Application = "Family-12345"
ID = "0003.00" (Weight in lbs)
Descr = "15" (15 units authorized at this weight)
```

This allows weight-based product families without schema changes.

---

## Major Use Case Categories

### 1. **Configuration Storage** (Product-specific settings)

- SellByOffset - Sell-by date offset per product
- MaximumKillDateOffset - Kill date limits
- Various timing and validation parameters

### 2. **Dynamic Lists** (Dropdown/validation values)

- ValidMachineIDs - Allowed machine identifiers
- ValidTubIDs - Allowed container/tub identifiers
- ProcessingModes - Available production modes

### 3. **Context Tracking** (Remember what user did last)

- LastsForProduct - Last value used per product
- LastsForLogin - Last value used in session
- LastsFor - General last-used tracking

### 4. **Mapping & Relationships** (Connect two things)

- ZuluDeptsShift - Shifts to departments
- PVPProductShift - Production lines to shifts
- OssidOrdNum - Orders to scale/printer
- TrayPrinterPlantID - Plants to printer configurations

### 5. **Counters & Metrics** (Track usage)

- Counter-Goal - Goal instance counters
- Counter-Production - Production batch counters

### 6. **Multi-valued Data** (Alternative to separate tables)

- Family-{ProductCode} - Weight tiers for product families
- FamilyItem-{code} - Item family alternatives

### 7. **External System Integration** (Configuration)

- WebSvcMii - IP addresses and paths to web services
- MacD - Barcode to product mapping

---

## Why This Design Is Brilliant

### 1. **No Schema Migrations**

New types of configuration don't require database changes. Just add a new Application type.

### 2. **Client-Specific Configuration**

Different factories, plants, or shifts can have completely different configurations without code changes.

### 3. **Runtime Flexibility**

Changes take effect immediately via the UI without requiring:

- Database migration
- Code recompilation
- Service restart
- Deployment

### 4. **Queryable History**

You can easily query what configurations were in use: `FOR EACH CrossRef WHERE Application MATCHES "LastsFor*":`

### 5. **Minimal Data Footprint**

Compared to having 35+ different tables, the CrossRef approach is extremely space-efficient.

### 6. **Self-Documenting**

The Application name is descriptive. "LastsForProduct" clearly means "stores last-used values per product."

---

## The Trade-Off

**Flexibility vs. Strong Typing:**

- ✅ Easy to add new configuration types
- ❌ No database constraints to validate format
- ❌ Type (text, number, date) is not enforced
- ❌ IDE autocomplete cannot help (unlike strongly-typed tables)

**Mitigation:**

Developers must:

1. Document expected format in comments (like the ones in this guide)
2. Add validation in code: `vDays = INTEGER(CrossRef.Descr) NO-ERROR`
3. Use consistent naming conventions for Application types

---

## How to Use CrossRef in Code

### Reading a Value

```progress
FIND CrossRef WHERE
    CrossRef.Application = "SellByOffset" AND
    CrossRef.ID = cProductCode
    NO-ERROR.
IF AVAILABLE(CrossRef) THEN DO:
    vSellByDays = INTEGER(CrossRef.Descr) NO-ERROR.
END.
```

### Writing/Updating a Value

```progress
FIND CrossRef WHERE
    CrossRef.Application = "MyConfig" AND
    CrossRef.ID = "MyKey"
    EXCLUSIVE-LOCK NO-ERROR.
IF NOT AVAILABLE(CrossRef) THEN DO:
    CREATE CrossRef.
    ASSIGN CrossRef.Application = "MyConfig"
           CrossRef.ID = "MyKey".
END.
ASSIGN CrossRef.Descr = "MyValue".
```

### Querying Multiple Values

```progress
FOR EACH CrossRef WHERE
    CrossRef.Application = "LastsForProduct" AND
    CrossRef.ID MATCHES "ProductKillDateRange-*"
    NO-LOCK:
    MESSAGE CrossRef.ID SKIP CrossRef.Descr VIEW-AS ALERT-BOX.
END.
```

---

## How Users Interact with CrossRef

### UI Access

- **Menu:** Configure → Edit Cross Reference
- **Program:** [comm/crossref.w](../comm/crossref.w)
- **Interface:** Browse all, Find by Application/ID, Create/Edit/Delete

### For Non-Technical Users

The CrossRef browser provides a GUI to:

- Add new valid machine IDs without contacting developers
- Update product-specific settings instantly
- Modify shift-to-department mappings
- Test new configurations before deployment

### For Developers

Use CrossRef when you need:

- Configuration that varies per product/shift/plant
- A list that changes frequently
- Default values that track user behavior
- Mappings between system entities

---

## Final Summary

**Cross References = A universal configuration table that trades strict typing for unlimited flexibility.**

The CrossRef table is the secret sauce that allows SLC to:

- Support multiple customers with different needs
- Allow configuration changes without code deployment
- Add new feature types without schema migration
- Empower non-developers to configure the system

It's a brilliant design pattern that you'll see throughout enterprise applications. OpenEdge (Progress) is particularly suited to this pattern because its dynamic nature makes the lack of strictly-typed configuration less painful than in typed languages.
