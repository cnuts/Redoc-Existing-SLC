

[Understanding Cross References](#understanding-cross-references-in-detail)
# Cross References (CrossRef Table)

This document lists all cross reference types used in the SLC system. The **CrossRef** table provides a flexible lookup mechanism for storing configuration, tracking, and mapping data using an Application/ID/Description triplet.

## Table Structure

The CrossRef table has three fields:

- **Application** - Category/type of cross reference (primary key component)
- **ID** - Identifier within the application (primary key component)
- **Descr** - Description or value associated with the Application/ID pair

**Indexes:**

- `UniqueAppCommIdx` (Application, ID) - Primary unique index
- `ApplicationIDX` - Application alone (for range queries)
- `IDIdx` - ID alone (for lookups)

---

## Cross Reference Categories

### Product-Related Applications

| Application Name            | Purpose                                       | ID Examples                          | Description Field Usage                      |
| --------------------------- | --------------------------------------------- | ------------------------------------ | -------------------------------------------- |
| **Product-MFGID**           | Manufacturing ID cross-reference for products | 5-digit product code (e.g., "12345") | 7-digit manufacturing ID                     |
| **Product-UCCGTINPackType** | UCC/GTIN pack type information                | Product code                         | Pack type designation (e.g., "9", "display") |
| **Product**                 | General product cross-reference data          | Product code                         | InkjetFormat or other product-specific data  |

**Used By:**

- [viewers/v-products.w](../viewers/v-products.w) - Product maintenance UI
- [queries/q-products.w](../queries/q-products.w) - Product queries

---

### Processing & Operations

| Application Name              | Purpose                           | ID Examples              | Description Field Usage            |
| ----------------------------- | --------------------------------- | ------------------------ | ---------------------------------- |
| **ProcessingModes**           | Available processing mode options | (empty or static)        | Comma-separated list of mode names |
| **ProcessingMode**            | Default processing mode selection | (empty)                  | Selected mode identifier           |
| **ProcessingModesOptionName** | Processing mode display names     | Mode-specific identifier | Human-readable mode name           |
| **ProcessingLine**            | Production line configuration     | Line identifier          | Line display name/configuration    |

**Used By:**

- [proc-modes/default-set-proc-mode-menu.w](../proc-modes/default-set-proc-mode-menu.w) - Mode selection interface
- [print/op-set-process-line-browse.w](../print/op-set-process-line-browse.w) - Line configuration

---

### Track & Trace (Lasts For)

These applications track the last values used for different contexts, enabling smart defaults and recall.

| Application Name    | Purpose                            | ID Format                         | Description Field Usage          |
| ------------------- | ---------------------------------- | --------------------------------- | -------------------------------- |
| **LastsForProduct** | Last used values per product       | "ProductKillDateRange-{prodcode}" | Last value used for that product |
| **LastsForLogin**   | Last used values per login session | "ProductKillDateRange-{prodcode}" | Last value used in session       |
| **LastsFor**        | General last-used tracking         | Process-specific identifiers      | Last used value                  |

**Used By:**

- [print/op-set-killdate-range.w](../print/op-set-killdate-range.w) - Kill date range setup
- [print/op-set-dates.w](../print/op-set-dates.w) - Date setting operations
- [print/select-mfgord.w](../print/select-mfgord.w) - Manufacturing order selection

**Example Usage Pattern:**
When a user sets a kill date range for Product A, the value is stored in CrossRef with:

- Application = "LastsForProduct"
- ID = "ProductKillDateRange-AAAAA"
- Descr = (the actual date range that was used)

Next time the same product loads, that value can be recalled as a default.

---

### Counters & Metrics

| Application Name       | Purpose                 | ID Examples          | Description Field Usage |
| ---------------------- | ----------------------- | -------------------- | ----------------------- |
| **Counter-Goal**       | Goal target counters    | Goal-specific IDs    | Counter value           |
| **Counter-Production** | Production run counters | Production batch IDs | Counter value           |

**Used By:**

- [print/weigh.w](../print/weigh.w) - Weighing operations
- [lib/clearlogs.p](../lib/clearlogs.p) - Log clearing and maintenance

---

### Shift & Department

| Application Name    | Purpose                                          | ID Format                                       | Description Field Usage              |
| ------------------- | ------------------------------------------------ | ----------------------------------------------- | ------------------------------------ |
| **ZuluDeptsShift**  | Department/shift assignment for Zulu capture     | Shift identifier (numeric, e.g., "1", "2", "3") | Department code or department list   |
| **PVPProductShift** | Production line/shift mapping for PVP operations | Shift code                                      | Product/line configuration for shift |

**Used By:**

- [sobjects/s-modify.w](../sobjects/s-modify.w) - Modify setup
- [print/weigh.w](../print/weigh.w) - Weight operations
- [sobjects/s-mainmenu.w](../sobjects/s-mainmenu.w) - Main menu initialization

**Example:**

- Application = "ZuluDeptsShift"
- ID = "1" (Shift 1)
- Descr = "Dept1|Dept2|Dept3" (pipe-separated department list)

---

### Printer & Output

| Application Name        | Purpose                               | ID Format             | Description Field Usage    |
| ----------------------- | ------------------------------------- | --------------------- | -------------------------- |
| **TrayPrinterPlantID**  | Tray printer configuration per plant  | Plant identifier      | Printer configuration data |
| **OssidOrdNum**         | Order number to scale/printer mapping | Order number          | Scale/printer assignment   |
| **WPL-Line-This-Scale** | Weighing/Print Line assignment        | Scale/line identifier | Current line assignment    |

**Used By:**

- [print/op-set-tray-printer.w](../print/op-set-tray-printer.w) - Printer setup
- [print/set-up-ossid-order.w](../print/set-up-ossid-order.w) - OSSID (Order/Scale/Serial ID) setup
- [print/update-ossid-current-order.p](../print/update-ossid-current-order.p) - Order updates

---

### Validation Lists

| Application Name    | Purpose                         | ID Format         | Description Field Usage                             |
| ------------------- | ------------------------------- | ----------------- | --------------------------------------------------- |
| **ValidTubIDs**     | Valid tub/container identifiers | (typically empty) | Comma-separated or pipe-separated list of valid IDs |
| **ValidMachineIDs** | Valid machine identifiers       | (typically empty) | Comma-separated or pipe-separated list of valid IDs |

**Used By:**

- [print/tub-id.w](../print/tub-id.w) - Tub ID validation
- [print/machine-id.w](../print/machine-id.w) - Machine ID validation

**Example:**

- Application = "ValidMachineIDs"
- ID = (empty)
- Descr = "M001,M002,M003,M004"

---

### Barcode & Scanning

| Application Name | Purpose                                | ID Format                 | Description Field Usage |
| ---------------- | -------------------------------------- | ------------------------- | ----------------------- |
| **MacD**         | McDonald's product barcode translation | 14-character GTIN barcode | Product code or mapping |
| **ScanMacD**     | McDonald's alternative scanning method | Barcode value             | Product code or mapping |

**Used By:**

- [print/scan-shapiro.p](../print/scan-shapiro.p) - Barcode scanning
- [print/scan-shapiro-macd.p](../print/scan-shapiro-macd.p) - McDonald's specific scanning

---

### Dynamic Pattern Applications

These applications use dynamic ID patterns based on runtime data:

| Application Pattern      | Purpose                      | ID Pattern                                  | Description Field Usage      |
| ------------------------ | ---------------------------- | ------------------------------------------- | ---------------------------- |
| **Family-{ProductCode}** | Family product weight ranges | Weight (format: "0001.00", "0002.00", etc.) | Authorized weight increments |
| **FamilyItem-{code}**    | Item family alternatives     | Item code                                   | Alternative item code        |

**Used By:**

- [print/case-family-print.p](../print/case-family-print.p) - Family case printing
- [print/item-family-print.p](../print/item-family-print.p) - Family item handling

**Example:**

- Application = "Family-12345"
- ID = "0001.00"
- Descr = "1" (authorized count at this weight)

---

### Date & Timing

| Application Name   | Purpose                              | ID Format                 | Description Field Usage         |
| ------------------ | ------------------------------------ | ------------------------- | ------------------------------- |
| **AutoLotEndTime** | Automatic lot end time configuration | Time identifier           | DateTime value in system format |
| **SellByOffset**   | Sell-by date offset from pack date   | Product/config identifier | Number of days offset           |

**Used By:**

- [print/get-sellby-offset.p](../print/get-sellby-offset.p) - Sell-by date calculation
- [print/auto-lot-end-time.p](../print/auto-lot-end-time.p) - Lot end time automation

**Example:**

- Application = "SellByOffset"
- ID = "12345" (product code)
- Descr = "30" (30 days from pack date)

---

### Quality & Rejection

| Application Name    | Purpose                        | ID Format                          | Description Field Usage          |
| ------------------- | ------------------------------ | ---------------------------------- | -------------------------------- |
| **RejectorSetting** | Reject mechanism configuration | "RejectTiming" or "RejectDuration" | Milliseconds for timing/duration |

**Used By:**

- [sobjects/s-rejectionsetup.w](../sobjects/s-rejectionsetup.w) - Rejection configuration UI
- [print/op-set-rejector.w](../print/op-set-rejector.w) - Rejector operation

**Entries:**

- Application = "RejectorSetting", ID = "RejectTiming", Descr = "1750" (milliseconds)
- Application = "RejectorSetting", ID = "RejectDuration", Descr = "1000" (milliseconds)

---

### System & Updates

| Application Name    | Purpose                              | ID Format           | Description Field Usage                  |
| ------------------- | ------------------------------------ | ------------------- | ---------------------------------------- |
| **UpdateAvailable** | Application update availability flag | "1" (true) or empty | Logical flag indicating update available |
| **Serial Upload**   | Serial number upload tracking        | Serial number       | Upload status or metadata                |

**Used By:**

- [sobjects/s-infobar.w](../sobjects/s-infobar.w) - Info bar status display
- [system/deleteserials.w](../system/deleteserials.w) - Serial deletion
- [lib/set-available-update.p](../lib/set-available-update.p) - Update check

---

### Debug Applications

Debug applications are used for troubleshooting and development diagnostics:

| Application Name     | Purpose                            | ID Format         | Description Field Usage      |
| -------------------- | ---------------------------------- | ----------------- | ---------------------------- |
| **Debug-WPL-Find**   | Debug data for WPL find operations | Operation context | Debug information/trace data |
| **Debug-WPL-Setup**  | Debug data for WPL setup           | Setup context     | Debug information/trace data |
| **Debug-WPL-Update** | Debug data for WPL updates         | Update context    | Debug information/trace data |

**Used By:**

- [print/find-ossid-current-order.p](../print/find-ossid-current-order.p) - Order finding logic
- [print/set-up-ossid-order.w](../print/set-up-ossid-order.w) - Order setup
- [print/update-ossid-current-order.p](../print/update-ossid-current-order.p) - Order updates

---

### Web Service Integration

| Application Name                        | Purpose                              | ID Format | Description Field Usage   |
| --------------------------------------- | ------------------------------------ | --------- | ------------------------- |
| **WebSvcMii**                           | Base MII web service configuration   | "Host"    | IP address of MII server  |
| **WebSvcMiiGetBatchesFromContainerID**  | Container ID batch retrieval service | "Path"    | Service API endpoint path |
| **WebSvcMiiGetPackDateBatch023**        | Pack date batch service (MSG023)     | "Path"    | Service API endpoint path |
| **WebSvcMiiGetBatchesFromLineProdCode** | Line product code batch retrieval    | "Path"    | Service API endpoint path |

**Used By:**

- [mii/webservices/bridge-get-batches-for-container-id.p](../mii/webservices/bridge-get-batches-for-container-id.p)
- [mii/webservices/bridge-get-order-sellby-date-msg023.p](../mii/webservices/bridge-get-order-sellby-date-msg023.p)
- [mii/webservices/bridge-get-batches-for-line-prodcode.p](../mii/webservices/bridge-get-batches-for-line-prodcode.p)

**Example Configuration:**

```
Application = "WebSvcMii"
ID = "Host"
Descr = "192.168.1.100"
Application = "WebSvcMiiGetBatchesFromContainerID"
ID = "Path"
Descr = "/services/GetBatchesFromContainerID"
```

---

## Cross Reference Usage Patterns

### Pattern 1: Configuration Storage

Store configuration values that vary by context:

```
Application = "SellByOffset"
ID = "{ProductCode}"
Descr = "{DateOffsetDays}"
```

### Pattern 2: Dynamic Lists

Store comma or pipe-separated values for populating dropdowns:

```
Application = "ValidMachineIDs"
ID = ""
Descr = "M001,M002,M003,M004"
```

### Pattern 3: Context-Aware Defaults (Lasts For)

Store the last used value for recall:

```
Application = "LastsForProduct"
ID = "ProductKillDateRange-AAAAA"
Descr = "{LastUsedKillDateRange}"
```

### Pattern 4: Shift/Department Mapping

Map shifts to departments or configurations:

```
Application = "ZuluDeptsShift"
ID = "{ShiftNumber}"
Descr = "Dept1|Dept2|Dept3"
```

### Pattern 5: Dynamic Family Ranges

Support product families with multiple weight tiers:

```
Application = "Family-{ProductCode}"
ID = "{Weight}"
Descr = "{Count}"
```

---

## Accessing Cross References

### In Progress Code

Use the `CrossRef` table directly:

```progress
FIND CrossRef WHERE
    CrossRef.Application = "SellByOffset" AND
    CrossRef.ID = cProductCode
    NO-ERROR.
IF AVAILABLE(CrossRef) THEN DO:
    vSellByDays = INTEGER(CrossRef.Descr).
END.
```

### Creating/Updating Cross References

```progress
FIND CrossRef WHERE
    CrossRef.Application = "MyApplication" AND
    CrossRef.ID = "MyID"
    EXCLUSIVE-LOCK NO-ERROR.
IF NOT AVAILABLE(CrossRef) THEN DO:
    CREATE CrossRef.
    ASSIGN CrossRef.Application = "MyApplication"
           CrossRef.ID = "MyID".
END.
ASSIGN CrossRef.Descr = cNewValue.
```

### UI for Cross Reference Maintenance

- [comm/crossref.w](../comm/crossref.w) - Main cross reference browser
- Accessed from Configuration Menu → Edit Cross Reference

---

## Performance Considerations

- The `UniqueAppCommIdx` (Application, ID) index ensures fast lookups
- Queries filtering by Application alone can use the `ApplicationIDX`
- The Descr field is not indexed; content searches require table scans
- Cross references are cached in memory where possible for frequently accessed configurations

---

## Summary Statistics

- **Total Unique Applications**: 35+ distinct types
- **Static Applications**: 32
- **Dynamic Pattern Applications**: 3+ (Family-_, FamilyItem-_)
- **Main Categories**: Products, Processing, Tracking, Metrics, Shifts, Output, Validation, Scanning, Timing, Quality, System, Debug, Web Services

