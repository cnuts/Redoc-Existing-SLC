---
Risk-Level: P0
Business-Rule-Id: BR-Modify-006
Deprecated: false
---

# Description

The Modify screen (s-modify.w) enforces minimum offset values when the user uses the BACK buttons for pack date, kill date, and variable offset. If the decremented value would go below the configured minimum, the change is reverted and the user is shown a message (ConfigMaxExceed or ConfigMinExceed). The limits are site/global: gMaxBackDate (pack date), gMinKillOffset (kill date), gMinVarOffset (variable offset).

# Source

- progress-SLC/sobjects/s-modify.w: btn-BackPackDate (vPackDateOffset - 1; if < gMaxBackDate revert and show ConfigMaxExceed); btn-BackKillDate (vKillDateOffset - 1; if < gMinKillOffset revert and show ConfigMinExceed); btn-BackVarOffset (vVariableOffset - 1; if < gMinVarOffset revert and show ConfigMaxExceed)
- Messages: PrintLabels.Modify.Msg.UpdatePDate, UpdateKdate, UpdateVarOffset; PrintLabels.Modify.Msg.ConfigMaxExceed, ConfigMinExceed

# Impacted Systems

s-modify.w, site/global config (gMaxBackDate, gMinKillOffset, gMinVarOffset), operator UI for pack/kill/variable date offsets.

# Traceability

- Pack date BACK: vPackDateOffset must remain >= gMaxBackDate (maximum back date); otherwise offset is restored and ConfigMaxExceed message is shown with gMaxBackDate.
- Kill date BACK: vKillDateOffset must remain >= gMinKillOffset; otherwise offset is restored and ConfigMinExceed message is shown.
- Var offset BACK: vVariableOffset must remain >= gMinVarOffset; otherwise offset is restored and ConfigMaxExceed message is shown.

# Assertions

- All three validations use a decrement-then-check pattern: value is decremented, then if below minimum the value is incremented back and the message is displayed.
- Display procedures (DisplayPackDate, DisplayKillDate, DisplayVariableDate) are run after the button logic so the screen reflects the (possibly reverted) value.

# Related Information

- [[BR-Modify-003-Modify-screen-permissions]]
