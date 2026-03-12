# SLC Application Glossary

A reference for **concepts**, **entities**, **algorithms** (named procedures/functions), **session globals**, **configuration and cross-reference terms**, and **abbreviations** used in the Scale Labeling Station (SLC) application.

---

## 1. Application and Product Terms

| Term                | Definition                                                                                                                                                                                                                                                                     |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **SLC**             | Scale Labeling Station. The application that weighs cases (and optionally items), validates weight, and prints labels; can run in Produce to Goal or Print Label mode.                                                                                                         |
| **CTS**             | Computerway Touch Screen.  Used in code/comments to refer to the system or session (e.g. Chickway section in .ini, session context).                                                                                                                                           |
| **Produce to Goal** | Mode where production is tied to a selected goal; serials carry GoalID and goal totals/status are updated on add/delete.                                                                                                                                                       |
| **Print Label**     | Mode where labels are printed without a goal (no GoalID).                                                                                                                                                                                                                      |
| Tare                | Tare weight refers to the weight of an empty container, vehicle, or packaging material, excluding the weight of any goods, cargo, or contents inside. It is used in logistics to calculate the net weight of cargo by subtracting the tare weight from the total gross weight. |
| **Work day (FM)**   | With False Midnight configured, the “day” for reporting runs from the FM time to the next FM time (e.g. 2 AM–2 AM), not calendar midnight.                                                                                                                                     |

---

## 2. Database Entities (Tables)

| Term | Definition |
|------|------------|
| **AppFunction** | Table of permission gate IDs (e.g. `ConfigMenu.SiteConfig.Update`). Each row defines one check used by **f-get-permission**. |
| **BatchOrder** | Links batch (BatchID, BatchDate, ProdCode) to goal selection; created when goal description is "Batch:*". |
| **CommDtl** | Communication detail (message body); child of CommHdr. |
| **CommHdr** | Communication header; host message queue and in-app inbox. |
| **ConfigOptionMaster** | Master list of site configuration options: OptionName, OptionDescr, OptionValue, AcceptableValues, OperatorEdits. |
| **CrossRef** | Flexible lookup: Application + ID + Descr. Used for Product-MFGID, Counter-Goal, Counter-Production, ZuluDeptsShift, ProcessingMode, LastsFor*, SellByOffset, etc. |
| **DateCode** | Table used for date-code-related data. |
| **Device** | Hardware endpoint: scale/indicator (input) or label printer (output). Model drives indicator driver; COM/NetworkID for output. |
| **ErrorMsgs** | Maps integer ReturnCode to ErrorText. |
| **Goals** | Production targets: target count/net/label weight, status (Ready, InProcess, Complete, etc.), link to Product (ProdCode). Totals updated when serials add/delete. |
| **Item** | Item master: ItemCode, descriptions, PackType, MinWgt, MaxWgt, StdWgt; used for item weights within a case. |
| **ItemModify** | Per-item runtime overrides (analogous to Modify for product). |
| **ItemSerial** | Item-level serial record; can carry GoalID; linked to Serial. |
| **Locals** | Display/local code mapping (e.g. product display codes). |
| **MailID** | Links operators/locations to CommSysID for message routing. |
| **Modify** | Per-product runtime overrides: TareToUse, PackDateOffset, KillDateOffset, Price, Lot, ModText1/ModText2 (NVP), CustName, OrdNum, pallet fields, etc. |
| **Msg** | Localized messages: LangCode, MsgId, Msg, MsgLen; used by get-Lang-Lbl / get-Lang-Msg. |
| **Operator** | User identity: Operator (login name, PK), Password, FirstName, LastName, LangCode. |
| **OperatorAppFunction** | Many-to-many (Operator, AppFunction.ID, Permitted). One row per operator per function; Permitted = true grants access. |
| **PalletDtl** | Pallet detail: serials/items on a pallet. |
| **PalletHdr** | Pallet header. |
| **Product** | Case-level product master: ProductCode (PK), descriptions, PackType, MinWgt, MaxWgt, StdWgt, label/date options, LabelFile, etc. |
| **ProductProcess** | Process steps per product: ProcessID, ProcessSequence, InputProcess/OutputProcess, Device; defines weigh/label flow. |
| **Serial** | One printed case: SerialNum, AddOrDel (A/D), ProductCode, weight fields, GoalID, PDNBatchID, Operator, UserBarString (NVPs), etc. Drives Totals and Goals updates. |
| **SiteConfigOption** | Per-site current value for each config option (OptionName, OptionValue). |
| **Totals** | Aggregates by Shift, Scale, PdnDate, ProductCode: LocalCount, LocalLabelWgt, LocalNetWgt, LocalTareWgt; updated when serials add/delete. |
| **UBDetail** | UserBar: per-character definition for a userbar format (UBField source, UBFieldCharLoc). |
| **UBExamples** | UserBar: setup guidance only. |
| **UBHeader** | UserBar: master definition (UBName A–Z, UBLength). |
| **ValidProcess** | Defines valid ProcessIDs used in ProductProcess. |

---

## 3. Concepts and Domain Terms

| Term                                   | Definition                                                                                                                                                                                     |
| -------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **AddOrDel**                           | Serial disposition: "A" = Add (production), "D" = Delete (e.g. reprint/cancel). Mandatory on Serial.                                                                                           |
| **Batch**                              | Production batch: BatchOrder rows; Serial.PDNBatchID; 4-digit BatchID in Serial NVPs; MII TT-Batch.                                                                                            |
| **Department**                         | Session-level value (gDepartment) chosen at login from ConfigOptionMaster "Valid Departments"; drives DEPTCODEDATE token and DeptCodeDate/DeptNumber NVPs. Distinct from Zulu Dept.            |
|                                        |                                                                                                                                                                                                |
| **False Midnight (FM)**                | Configurable time boundary (e.g. 2 AM) that defines the “work day” for pack/print date and optionally kill date (FalseMidnight / FalseMidnightKill).                                           |
| **Goal**                               | A production target (Goals row); status Ready/InProcess/Complete; selection by batch, goal list, or order; totals updated from serials.                                                        |
| Give Away                              | In this application Give Away (also Giveaway / Give-Away) is a weight metric: the difference between what was actually produced (net weight) and what is declared on the label (label weight). |
| **Indicator**                          | Scale/weight device; driver in indicators/*.p parses raw weight; weight used for min/max and label.                                                                                            |
| **Label**                              | Template (.lbl) with bracketed tokens; printlabel.p substitutes from Serial/Goals/Modify/NVPs; Set-Label-Wgt; output via Device.                                                               |
| **LimitBy**                            | Goals: how target is measured—Count, NetWgt, or LabelWgt.                                                                                                                                      |
| **Message**                            | In app: Msg (localization); CommHdr/CommDtl (host/inbox); ErrorMsgs (ReturnCode→text); Goals.AddMsg1/2/3.                                                                                      |
| **Modify**                             | Runtime overrides per product (table and concept); tt-Modify is session buffer; ZuluCapture stores ZuluDept in ModText1.                                                                       |
| **NVP**                                | Name-value pair; stored in strings (e.g. Serial.UserBarString, Modify.ModText1). Utilities: f-NVP-get-value, f-NVP-set-value.                                                                  |
| **PackType**                           | Product/Item pack type: C=Catch, F=Fixed, U=Unipak, K=Key-in, V=Variance, H=Hybrid. Drives weight and Set-Label-Wgt logic.                                                                     |
| **PDN**                                | Production (e.g. PDNBatchID, PdnDate, PdnOrdNum); sometimes “Production Down” or similar in host context.                                                                                      |
| **Print Date / Pack Date / Kill Date** | Dates on serial/label; adjusted by False Midnight and Modify offsets.                                                                                                                          |
| **ProductProcess**                     | Sequence of steps (get weight, print label, set batch, etc.) per product; engine in getnextprocess.p.                                                                                          |
| **Serial**                             | One physical case record (Serial table row) produced and optionally tied to a goal.                                                                                                            |
| **Site configuration**                 | ConfigOptionMaster (definitions) + SiteConfigOption (values); TT-Site-Value cache; f-get-session-parm / f-Get-Site-Value-TT.                                                                   |
| **UserBar**                            | Custom A–Z barcode formats defined by UBHeader/UBDetail; built by userbar.p; USERBARA–Z tokens on labels; Serial.UserBarString stores Barcode NVP and others.                                  |
| **ValidProcess**                       | Set of valid ProcessIDs for ProductProcess steps.                                                                                                                                              |
| **Zulu Dept**                          | Department value for meat/Zulu processing; stored in Modify.ModText1 as NVP ZuluDept or in CrossRef/SiteConfig by shift; distinct from login Department.                                       |

---

## 4. Named Algorithms (Procedures and Functions)

| Name | Location / Purpose |
|------|--------------------|
| **BuildFalseMidnightPDandKD** | get-sellby-offset.p: calls FalseMidnight in gh_CreateSerial to get adjusted pack/kill dates. |
| **Create-Modify** | weigh-create-modify.p: finds/creates Modify for gTTProductCode; applies GlobalKillDate, ZuluCapture/ZuluFromGlobal; buffer-copy to tt-Modify; optionally UpdateFromGoal. |
| **Create-SiteConfig** | lib/services.p (calls lib/create-sitecfg.p): creates SiteConfigOption (and ConfigOptionMaster if missing) for an option name with default value and description. |
| **CreateSerial** | lib/createserial.p: creates Serial record; runs FalseMidnight; sets UserBarString NVPs (DeptCodeDate, DeptNumber, Barcode, etc.); assigns GoalID, PDNBatchID; invokes updatetotals. |
| **FalseMidnight** | lib/createserial.p: given production time/date, returns FM-adjusted PackDate and KillDate using gFalseMidnight and gFalseMidnightKill (before/after noon logic). |
| **f-Get-Site-Value-TT** | lib/services.p: returns value from TT-Site-Value for option name; if missing, uses default and optionally creates SiteConfigOption via Create-SiteConfig. |
| **f-get-permission** | lib/services.p: returns true if gOperator=CFSUSER or gSecurityOverride, else true iff OperatorAppFunction exists for (gOperator, pi-AppFunction) with Permitted=true. |
| **f-get-session-parm** | lib/services.p: returns SiteConfigOption.OptionValue for parm name; optional type hint (LOG, INT, CHR). |
| **f-Rebuild-Site-Value-TT-From-Site** | lib/services.p: repopulates TT-Site-Value from SiteConfigOption. |
| **f-Set-Date-Per-False-Midnight** | lib/services.p: given date, time string, FM seconds, and NVP name, returns date adjusted per FM (used in modify/date UIs). |
| **f-Set-Site-Value-TT** | lib/services.p: updates or creates TT-Site-Value; optionally updates SiteConfigOption. |
| **GetNextSerialSeq** | lib/createserial.p: returns next serial number (and updates Counter-Production or Counter-Goal CrossRef, SerialResetDaily, etc.). |
| **Load-TT-Site-Values** | lib/services.p: fills TT-Site-Value from all SiteConfigOption rows (and registry SecurityOverride, Palletize). |
| **PrintLabel** | lib/printlabel.p: reads label template, substitutes tokens from CaseLabelSource/Product/Goals/NVPs, sends to output device. |
| **PrintLblInits** | lib/printlabel.p: initializes label run (paths, template resolution). |
| **Set-Label-Wgt** | print/set-label-wgt.p: given PackType and weights, returns label weight (e.g. Catch=scale weight, Fixed=StdWgt, Variance=clip to MinLabelWgt–MaxLabelWgt). |
| **Set-Labels** | Resolves first/quick/cancel label file paths for a product/goal. |
| **UpdateFromGoal** | weigh-create-modify.p: when gProduceToWO, copies goal fields (CasesPerPallet, CustomerName, OrdNum, PackDateOffset, UnitPrice, VarOffset, etc.) into tt-Modify. |
| **UpdateTotals** | lib/updatetotals.p (and updatetotals2.p): on serial add/delete, updates Totals (LocalCount, LocalLabelWgt, etc.) and Goals (Current*/AllScaleTotal*, status). |

---

## 5. Temp Tables and Buffers (Runtime Structures)

| Term | Definition |
|------|------------|
| **tt-Modify** | Temp table like Modify; populated by weigh-create-modify.p for current product (and goal overrides); passed into CreateSerial. |
| **tt-Product** | Temp table snapshot of Product for current product; may include family/CrossRef weight overrides. |
| **tt-ProductProcess** | Temp table of process steps for current product; built from ProductProcess for gTTProductCode. |
| **TT-Site-Value** | Temp table (OptionName, OptionValue) in lib/services.p; cache of SiteConfigOption; filled by Load-TT-Site-Values, updated by f-Set-Site-Value-TT. |
| **TT-Batch** | MII/webservices: batch data from container ID or line+prodcode. |
| **tt-CaseLabelSource** | Serial-like buffer used as data source for label token substitution in printlabel.p. |

---

## 6. Session and Processing Globals (Selected)

| Term | Definition |
|------|------------|
| **gDepartment** | Session department from login dropdown (ConfigOptionMaster "Valid Departments"); used for DEPTCODEDATE and DeptNumber NVPs. |
| **gFalseMidnight** | Pack/print date False Midnight time in seconds past midnight (from SiteConfigOption FalseMidnight). |
| **gFalseMidnightKill** | Kill date False Midnight in seconds (from SiteConfigOption FalseMidnightKill). |
| **gGoalID** | Current goal’s GoalID (Goals.GoalID). |
| **gGoalsRowid** | Rowid of current Goals record. |
| **gOperator** | Current operator login name (Operator.Operator). |
| **gPDNBatchID** | Current batch ID when producing to a batch goal (Serial.PDNBatchID). |
| **gProduceToWO** | Logical: true when producing to a work order/goal. |
| **gSecurityOverride** | When true, f-get-permission always returns true; from .ini Chickway.SecurityOverride. |
| **gShift** | Current shift number. |
| **gTTProductCode** | Current product code; keys tt-Product, tt-Modify, tt-ProductProcess, Item temp tables. |

---

## 7. Configuration and Cross-Reference Terms

| Term | Definition |
|------|------------|
| **AcceptableValues** | ConfigOptionMaster: pipe-separated list of valid values for an option. |
| **ConfigOptionMaster** | Table of option definitions (OptionName, OptionDescr, OptionValue, AcceptableValues, OperatorEdits). |
| **Counter-Goal** | CrossRef application: counter per goal (Application=Counter-Goal, ID=Goals.GoalID). |
| **Counter-Production** | CrossRef application: production counter per date+shift+ProductCode (no goal). |
| **Create-SiteConfig** | Procedure that creates SiteConfigOption (and ConfigOptionMaster if absent) for an option. |
| **FalseMidnight / FalseMidnightKill** | SiteConfigOption names: time in HH:MM (or 24:00 = off) for pack/print and kill date boundaries. |
| **OperatorEdits** | ConfigOptionMaster: whether operators can edit this option in site config UI. |
| **SiteConfigOption** | Table of current option values (OptionName, OptionValue) per site. |
| **Valid Departments** | ConfigOptionMaster OptionName: AcceptableValues list for login department dropdown. |
| **ValidShifts** | ConfigOptionMaster OptionName: AcceptableValues list for valid shift at login. |
| **ZuluCapture** | Site config: whether Zulu Dept is captured and stored in Modify.ModText1 (NVP ZuluDept). |
| **ZuluDeptsShift** | CrossRef application: valid Zulu dept list per shift (ID = shift number, Descr = list). |
| **ZuluFromGlobal** | Site config: use global Zulu Dept value for all products (ZuluGlobalDeptShift + shift). |
| **ZuluGlobalDeptShift1/2/3** | SiteConfigOption: global Zulu Dept value for shift 1, 2, or 3. |

---

## 8. Label and UserBar Terms

| Term | Definition |
|------|------------|
| **BARCODE** | Label token: value from Serial.UserBarString NVP "Barcode" (from CreateBarCode). |
| **DeptCodeDate** | NVP and token: gDepartment + 2-digit (DAY(PackDate)*2); also in UserBarString. |
| **DeptNumber** | NVP: gDepartment only; stored in Serial.UserBarString. |
| **DEPTCODEDATE** | Label token: same as DeptCodeDate. |
| **USERBARA–USERBARZ** | Label tokens: userbar string for format A–Z, built at print time by userbar.p. |
| **UserBarString** | Serial field: name-value pair string (Barcode, DeptCodeDate, DeptNumber, ZuluDept, BatchID, Counter, etc.). |

---

## 9. Acronyms and Abbreviations

| Term    | Definition                                                                                                                   |
| ------- | ---------------------------------------------------------------------------------------------------------------------------- |
| **AD**  | Active Directory (e.g. group-based login).                                                                                   |
| DTR     | Data Terminal Ready.  RS-232 control signal                                                                                  |
| **FM**  | False Midnight.                                                                                                              |
| GTIN    | Global Trade Item Number - unique internationally recognized 8-14 digit identifier.  Often ended as barcodes like UPC or EAN |
| **ICI** | Interface/communication (e.g. ICICTSID).                                                                                     |
| **MII** | Manufacturing Integration (e.g. MII batch webservices).                                                                      |
| **NVP** | Name-value pair.                                                                                                             |
| **PDN** | Production (PDNBatchID, PdnDate, PdnOrdNum, etc.).                                                                           |
| **PVP** | Product/process variant (e.g. PVPProducts, PVPProductShift).                                                                 |
| **SLC** | Scale Labeling Station.                                                                                                      |
| **TT**  | Temp table (e.g. TT-Site-Value, tt-Modify).                                                                                  |
| **UB**  | UserBar (UBHeader, UBDetail, UBExamples).                                                                                    |
| **WPL** | Weigh/Print Line or related (e.g. WPL SQL API).                                                                              |
| **WO**  | Work order (e.g. gProduceToWO = produce to work order/goal).                                                                 |


