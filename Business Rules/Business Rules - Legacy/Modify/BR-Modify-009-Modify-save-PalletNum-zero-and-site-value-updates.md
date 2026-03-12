---
Risk-Level: P0
Business-Rule-Id: BR-Modify-009
Deprecated: false
---

# Description

When saving from the Modify screen (s-modify.w), if PalletTracking is false, Modify.PalletNum is set to '0'. The save also updates site config values: GlobalKillDate is set from the computed kill date (v-TodayAtLocalInit + vKillDateOffset). When GlobalLotProgram is set to the global-lot program, Modify.Lot is written to the GlobalLotNumber site value. When ZuluCapture and ZuluFromGlobal are true, the ZuluGlobalDeptShift value for the current shift (ZuluGlobalDeptShift + STRING(gShift)) is updated from the selected Zulu dept. Change logging is performed via create-log-from-buffers.p with reason "Modify Maint".

# Source

- progress-SLC/sobjects/s-modify.w: assign Modify.PalletTrack = (filPallet = "yes"); Modify.PalletNum = (if not Modify.PalletTrack then '0' else Modify.PalletNum); f-Set-Site-Value-TT GlobalKillDate, GlobalLotNumber (when v-SetGlobalLot), ZuluGlobalDeptShift+shift (when ZuluCapture and ZuluFromGlobal); run lib/create-log-from-buffers.p "Modify Maint"

# Impacted Systems

Modify table (PalletNum, PalletTracking), SiteConfigOption (GlobalKillDate, GlobalLotNumber, ZuluGlobalDeptShift+shift), s-modify.w, create-log-from-buffers.p.

# Traceability

- PalletNum = '0' when not tracking ensures a sentinel value when pallet tracking is off.
- Site value updates persist the user's kill date, global lot, and global Zulu dept per shift for reuse (e.g. in Create-Modify).

# Assertions

- Modify.PalletNum is explicitly set to '0' when PalletTracking (PalletTrack in code) is false; otherwise existing PalletNum is retained.
- GlobalKillDate site value is updated on every Modify save from the computed kill date.
- create-log-from-buffers is called with KeyVal/KeyVal1/KeyVal2 = Modify.ProductCode, Reason = "Modify Maint".

# Related Information

- [[BR-Modify-002-GlobalKillDate-and-ZuluCapture]]
- [[BR-Modify-003-Modify-screen-permissions]]
