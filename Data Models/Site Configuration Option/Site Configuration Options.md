
*Note: extracted from ABL source of the SLC*
## Configuration Tables

### SiteConfigOption Table
Contains the active configuration settings for the current site. Each record has:
- **OptionName** - Name of the configuration option (primary key)
- **OptionValue** - Current value of the option

### ConfigOptionMaster Table
Contains the master definitions of all configuration options. Each record has:
- **OptionName** - Name of the configuration option (primary key)  
- **OptionDescr** - Description of what the option controls
- **OptionValue** - Default or recommended value
- **AcceptableValues** - Pipe-separated list of valid values (|)
- **OperatorEdits** - Boolean flag indicating if operators can edit this option

## Accessing Configuration Options

Configuration options can be accessed through:
1. **UI Browser**: [b-sitecfg.w](../browsers/b-sitecfg.w) - Browser for viewing/editing site configuration
2. **Dialog**: [d-sitecfgfind.w](../dialogs/d-sitecfgfind.w) - Find/search dialog for configuration
3. **SQL/API**: Direct database access to SiteConfigOption and ConfigOptionMaster tables
4. **Functions**: `f-Get-Site-Value-TT()` in [lib/services.p](../lib/services.p) for programmatic access

## Notes

- Configuration options are case-sensitive
- Multiple values are separated by pipe characters (|)
- The system will automatically create missing configuration options with default values on first use
- Some options have restrictions on who can edit them (OperatorEdits flag)
- Changes to configuration typically take effect immediately or after service restart


# Site Configuration Options

This document lists all **118 available configuration options** from the **ConfigOptionMaster** table in the SLC system. These options are stored in the **SiteConfigOption** table at runtime and managed through the **ConfigOptionMaster** table.

## Complete Configuration Options List (Alphabetically)

| Option Name                  | Description                                                               | Default Value              | Acceptable Values                      | Operator Editable |
| ---------------------------- | ------------------------------------------------------------------------- | -------------------------- | -------------------------------------- | ----------------- |
| AIOffset1Date                | Offset1 base for Serial/Barcode/Label                                     | Production                 | Production\|Kill\|Now                  | Yes               |
| AllowPVPProducts             | Allow PVP Products                                                        | NO                         | YES\|NO                                | Yes               |
| AllScaleTotals               | All Scale Totals                                                          | (empty)                    | (empty)                                | Yes               |
| BatchOrder Clear Days        | Delete Batch Orders if older than date                                    | 7                          | 7                                      | Yes               |
| CEIA-CtrLoggedRuns           | CEIA Counter Logged Runs                                                  | (empty)                    | (empty)                                | Yes               |
| CEIA-LastPSGroup             | CEIA Last PS Group                                                        | (empty)                    | (empty)                                | Yes               |
| CEIA-LastPSGSucceeded        | CEIA Last PS Group Succeeded                                              | (empty)                    | (empty)                                | Yes               |
| Clear-Goals-Days             | Delete Goals if older than date, complete or canceled                     | 2                          | 2                                      | Yes               |
| Clear-MfgOrd-Days            | Delete MfgOrd if older than date and not Open                             | 7                          | 7                                      | Yes               |
| CountUpThreshold             | Count Up Threshold                                                        | (empty)                    | (empty)                                | Yes               |
| CurrentFiscalStartDate       | Current Fiscal MDY Perdue                                                 | 04/01/15                   | 04/01/15\|03/30/15\|03/31/14           | Yes               |
| Debug-SerialSeq              | Debug messages                                                            | no                         | yes\|no                                | No                |
| DebugScanner                 | Debug Scanner                                                             | (empty)                    | (empty)                                | Yes               |
| Enable Override Weights      | Enable Override Weights                                                   | NO                         | YES\|NO                                | Yes               |
| EnableSampleLabel            | Enable Sample Label                                                       | NO                         | YES\|NO                                | Yes               |
| ERPPlantID                   | ERP Plant ID                                                              | (empty)                    | (empty)                                | Yes               |
| FalseMidnight                | Adjust production date around midnight for pack date                      | (empty)                    | (empty)                                | Yes               |
| FalseMidnightKill            | Adjust production date around midnight for kill date                      | (empty)                    | (empty)                                | Yes               |
| FalseMidnightLot             | FalseMidnight Lot Time                                                    | 00:00                      | 00:00                                  | Yes               |
| FalseMidnight Set By Server  | FalseMidnight Set By Server                                               | YES                        | YES\|NO                                | Yes               |
| GlobalLotNumber              | Global Lot Number                                                         | (empty)                    | (empty)                                | Yes               |
| GlobalLotProgram             | Global Lot Program                                                        | (empty)                    | (empty)                                | Yes               |
| Goal Select By Date          | Goal Select By Date                                                       | YES                        | YES\|NO                                | Yes               |
| Goals-Purge-Days             | Delete Goals if older than date, disregard status                         | 7                          | 7\|14                                  | Yes               |
| GradientAll                  | Gradient All                                                              | (empty)                    | (empty)                                | Yes               |
| HostType                     | Host Type Configuration                                                   | (empty)                    | (empty)                                | Yes               |
| ICICTSID                     | ICIC TS ID                                                                | (empty)                    | (empty)                                | Yes               |
| IntervalMinsRefreshSerialnum | Interval Minutes Refresh Serial Number                                    | (empty)                    | (empty)                                | Yes               |
| KillDateRangeMaxDaysBack     | Max days can backdate for the range                                       | 5                          | 5                                      | Yes               |
| KillDateRangeMaxDaysBackDate | Max days can backdate for the range                                       | 5                          | 5                                      | Yes               |
| Languages                    | Available language options                                                | (empty)                    | (empty)                                | No                |
| LastProdCodeSelected         | Last Product Code Selected                                                | (empty)                    | (empty)                                | Yes               |
| LoginForceShiftEntry         | Force Shift Entry on Login                                                | (empty)                    | (empty)                                | Yes               |
| LoginPort                    | Login Port                                                                | (empty)                    | (empty)                                | Yes               |
| LotKeyIn                     | Modify screen key in lot fields                                           | No                         | Yes\|No                                | Yes               |
| LotProductionDate            | Use FalseMidnightLot Date as Lot                                          | False                      | False\|True                            | Yes               |
| MachineIDValidation          | Machine ID Validation                                                     | (empty)                    | (empty)                                | Yes               |
| MaximumBackDate              | Maximum Days to Backdate PDN                                              | (empty)                    | (empty)                                | Yes               |
| MaximumForwardDate           | Maximum Days to Forward Date PDN                                          | (empty)                    | (empty)                                | Yes               |
| MaximumKillDateOffset        | Maximum Kill Date Offset Days                                             | (empty)                    | (empty)                                | Yes               |
| Message Debug                | Message Debug                                                             | NO                         | YES\|NO                                | Yes               |
| MII-IgnoreModifyOffsets      | MII Ignore Modify Offsets                                                 | (empty)                    | (empty)                                | Yes               |
| MII-PackDate                 | MII Pack Date                                                             | (empty)                    | (empty)                                | Yes               |
| MII-PackOrder                | MII Pack Order                                                            | (empty)                    | (empty)                                | Yes               |
| MII-ProcessOrder             | MII Process Order                                                         | (empty)                    | (empty)                                | Yes               |
| MII-SellByDate               | MII Sell By Date                                                          | (empty)                    | (empty)                                | Yes               |
| MinimumKillDateOffset        | Minimum Kill Date Offset Days                                             | (empty)                    | (empty)                                | Yes               |
| ModeProdCodeDefault          | Proc Modes Default Product Code                                           | 99999                      | 99999                                  | Yes               |
| NTPServer                    | NTP Server Address                                                        | (empty)                    | (empty)                                | Yes               |
| Operator                     | Current Operator                                                          | (empty)                    | (empty)                                | Yes               |
| PDNHostName                  | PDN Host Name                                                             | (empty)                    | (empty)                                | Yes               |
| PDNVerificationIP            | PDN Verification Address                                                  | (empty)                    | (empty)                                | Yes               |
| PDNVerificationPort          | PDN Verification Port                                                     | 23                         | 23                                     | Yes               |
| PDNVerificationServerPort    | PDN Verification Server Port                                              | 23                         | 23                                     | Yes               |
| PgmSpecialProcessing         | Pgm for switching processing modes                                        | Default-Set-Proc-Mode-Menu | Dynamic list from proc-modes folder    | Yes               |
| PlantID                      | Plant ID                                                                  | (empty)                    | Populated from import or configuration | No                |
| PriorFiscalStartDate         | Prior Fiscal MDY Perdue                                                   | 03/31/14                   | 04/01/15\|03/30/15\|04/01/13\|03/31/14 | Yes               |
| ProcessDownLoads             | Process downloads from server                                             | Yes                        | YES\|NO                                | Yes               |
| ProcessingMode               | CrossRef entry for Default Product Process                                | Default                    | Default                                | Yes               |
| ProcModeNextScreen           | After ProcMode change, go here                                            | (empty)                    | \|LocalCodes\|Goals                    | Yes               |
| PVPProducts                  | PVP Products                                                              | (empty)                    | (empty)                                | Yes               |
| ResendSerialsRetryMins       | Resend Serials Retry Minutes                                              | (empty)                    | (empty)                                | Yes               |
| SampleLabelQty               | Sample Label Quantity                                                     | 1                          | 1\|2                                   | Yes               |
| SampleLabelRequired          | Sample Label Required                                                     | NO                         | YES\|NO                                | Yes               |
| SameWeightsDelta             | Consecutive weights range: +/-                                            | 0                          | 0                                      | Yes               |
| SameWeightsCheck             | Check consecutive weights for in-range                                    | NO                         | NO\|YES                                | Yes               |
| SameWeightsToCheck           | Number of Consecutive Wgts In Range To Trigger Message                    | 3                          | 3                                      | Yes               |
| ScaleComPort                 | Scale Communication Port                                                  | (empty)                    | (empty)                                | Yes               |
| ScaleDataBits                | Scale Data Bits                                                           | (empty)                    | (empty)                                | Yes               |
| ScaleFlowControl             | Scale Flow Control                                                        | (empty)                    | (empty)                                | Yes               |
| ScaleID                      | Unique Scale Number                                                       | 99                         | (empty)                                | Yes               |
| ScaleParity                  | Scale Parity                                                              | (empty)                    | (empty)                                | Yes               |
| ScaleSpeed                   | Scale Speed (Baud Rate)                                                   | (empty)                    | (empty)                                | Yes               |
| ScaleStopBits                | Scale Stop Bits                                                           | (empty)                    | (empty)                                | Yes               |
| ScaleTimeOut                 | Scale Time Out                                                            | (empty)                    | (empty)                                | Yes               |
| ScannerEOM                   | Scanner END OF MESSAGE delimiter                                          | (carriage return)          | (carriage return)\|)                   | Yes               |
| Serial-Date                  | Date used in serial date fields                                           | (today's date)             | (empty)                                | No                |
| Serial-Time                  | Time used in serial time fields                                           | (current time)             | (empty)                                | No                |
| SerialNVPs                   | Serial Name-Value Pairs                                                   | (empty)                    | (empty)                                | Yes               |
| SerialResetDaily             | Serial to 1 on new day                                                    | yes                        | yes\|no                                | Yes               |
| SerialVerification           | Enable Serial Verification                                                | No                         | Yes\|No                                | Yes               |
| SLC-Version                  | SLC Version Number                                                        | (current version)          | (empty)                                | No                |
| TemporaryADUsers             | Temporary Active Directory Users                                          | (empty)                    | (empty)                                | Yes               |
| TextWeightField              | Field to use when Text Weight met                                         | Text1000                   | Text1000                               | Yes               |
| TgtCompDateTime Warn Level   | Target Completion Warning Level                                           | 60                         | 15\|30\|45\|60\|75\|90                 | Yes               |
| TrayPrinterSetupTimeoutMS    | Error Timeout in MS                                                       | 5000                       | 5000                                   | Yes               |
| TubIDValidation              | Tub ID Validation                                                         | (empty)                    | (empty)                                | Yes               |
| UpdateTotDebug               | Update Totals Debug                                                       | (empty)                    | (empty)                                | Yes               |
| UploadProdTotals             | Upload Production Totals                                                  | NO                         | YES\|NO                                | Yes               |
| UploadSerials                | Upload Serials                                                            | Yes                        | Yes\|No                                | Yes               |
| UseSerialNum8                | Use 8 Digit UniqueNum counter for serial derivation instead of JulianDate | no                         | yes\|no                                | Yes               |
| ValidShifts                  | Valid Shift Configuration                                                 | (empty)                    | (empty)                                | Yes               |
| WPL-SQL-API-IP               | WPL SQL API IP Address                                                    | CFSSERVER                  | CFSSERVER                              | Yes               |
| WPL-SQL-API-Port             | WPL SQL API Port                                                          | 49000                      | 49000                                  | Yes               |
| WPL-SQL-Broker               | WPL SQL Broker Service                                                    | wplbrkr                    | wplbrkr                                | Yes               |
| WPL-SQL-Broker-Hostname      | WPL SQL Broker Hostname                                                   | 1.1.1.23                   | 1.1.1.23                               | Yes               |
| WPL-SQL-PassWord             | WPL SQL PassWord                                                          | mask04                     | mask04                                 | Yes               |
| WPL-SQL-UnixHostName         | SQL UnixHostName for Schema                                               | 1.1.1.120                  | 1.1.1.120                              | Yes               |
| WPL-SQL-UnixService          | SQL Unix Service                                                          | wplsvr                     | wplsvr                                 | Yes               |
| WPL-SQL-Use-API              | WPL SQL Use API                                                           | NO                         | YES\|NO                                | Yes               |
| WPL-SQL-UserName             | WPL SQL UserName                                                          | cfs                        | cfs                                    | Yes               |
| ZuluCapture                  | Serial Contains Zulu                                                      | NO                         | NO\|YES                                | Yes               |
| ZuluFromGlobal               | Zulu From Global Value                                                    | NO                         | NO\|YES                                | Yes               |
| ZuluGlobalDeptShift1         | Zulu Dept value for Shift 1                                               | (empty)                    | (empty)                                | Yes               |
| ZuluGlobalDeptShift2         | Zulu Dept value for Shift 2                                               | (empty)                    | (empty)                                | Yes               |
| ZuluGlobalDeptShift3         | Zulu Dept value for Shift 3                                               | (empty)                    | (empty)                                | Yes               |

### SiteConfigOption Table

Contains the active configuration settings for the current site. Each record has:

- **OptionName** - Name of the configuration option (primary key)
- **OptionValue** - Current value of the option

### ConfigOptionMaster Table

Contains the master definitions of all configuration options. Each record has:

- **OptionName** - Name of the configuration option (primary key)
- **OptionDescr** - Description of what the option controls
- **OptionValue** - Default or recommended value
- **AcceptableValues** - Pipe-separated list of valid values (|)
- **OperatorEdits** - Boolean flag indicating if operators can edit this option

## Accessing Configuration Options

Configuration options can be accessed through:

1. **UI Browser**: [b-sitecfg.w](../browsers/b-sitecfg.w) - Browser for viewing/editing site configuration
2. **Dialog**: [d-sitecfgfind.w](../dialogs/d-sitecfgfind.w) - Find/search dialog for configuration
3. **SQL/API**: Direct database access to SiteConfigOption and ConfigOptionMaster tables
4. **Functions**: `f-Get-Site-Value-TT()` in [lib/services.p](../lib/services.p) for programmatic access

## Notes

- Configuration options are case-sensitive
- Multiple values are separated by pipe characters (|)
- The system will automatically create missing configuration options with default values on first use
- Some options have restrictions on who can edit them (OperatorEdits flag)
- Changes to configuration typically take effect immediately or after service restart
