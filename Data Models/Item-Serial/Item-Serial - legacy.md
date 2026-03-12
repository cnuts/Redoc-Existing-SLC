
**Source**: `slc.df`. One row per item label/weigh event (add or delete).

| Field | Type | Description |
|-------|------|-------------|
| **SerialNum** | character | Item serial number (unique per add). Mandatory. |
| **AddOrDel** | character (1) | A=Add, D=Delete (VALEXP). Mandatory. |
| **PrintDate**, **PrintTime** | date, integer | When label was printed. Mandatory. |
| **PackDate**, **KillDate** | date | Pack/kill dates. |
| **Shift**, **ScaleNumber** | integer | Shift and scale. |
| **Operator** | character | Operator at print. |
| **Sent** | logical | Sent to host. |
| **WgtUnits** | character | LB or KG. |
| **LabelWgt**, **NetWgt** | decimal | Label weight and net weight. |
| **ExtraTareWgt** | decimal | Extra tare weight. |
| **ItemCode** | integer | Item (00001–99999, VALEXP). |
| **Lot** | character | Lot. |
| **GoalID** | integer | Goal when produced to goal. |
| **UserBarString** | character | NVP string. |
| **Price** | decimal | Price. |
| **CustomerName** | character | Customer name. |
| **Order** | character | Order. |
| **ItemPostTareWgt**, **ItemPreTareWgt** | decimal | Item tare weights. |
| **TotalTareWgt** | decimal | Total tare. |
| **ProductCode** | integer | Product (case) code. |
| **CaseSerialNum** | character | Serial number of the case this item belongs to. |
| **PdnOrdNum**, **PdnOrdLineNum** | character, integer | PDN order. |

**Indexes**: ItemSerial (SerialNum, AddOrDel); CaseSerialNum; Goal; PackDate; PrintDateTime; ProductCode; Sent; Shift; etc.

**Relationship**: **ItemSerial** belongs to a **Serial** (case) via **CaseSerialNum** and **ProductCode**; references **Item** via **ItemCode** and **Goals** via **GoalID**.