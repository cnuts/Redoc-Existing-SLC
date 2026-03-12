---
Risk-Level: P0
Business-Rule-Id: BR-Modify-010
Deprecated: false
---

# Description

The Modify screen populates the Lot selection list (slLot) from ConfigOptionMaster for option GlobalLotNumber: AcceptableValues (pipe-separated entries) are added to the list; if the list does not contain 'None', 'None' is added first. If ConfigOptionMaster has no AcceptableValues for GlobalLotNumber, the screen shows MissingLotsTitle and MissingLots and does not build the list. When building modify fields, if Modify.Lot is not in AcceptableValues, the UI sets slLot to 'None' (or, when Key-In is available, shows Modify.Lot in the fill-in and RS-Lot to "Key"); otherwise slLot and Fil-Lot are set from Modify.Lot. Lot selection is thus restricted to configured values when the config is present.

# Source

- progress-SLC/sobjects/s-modify.w: FIND ConfigOptionMaster OptionName = 'GlobalLotNumber'; IF INDEX(AcceptableValues,'None')=0 THEN slLot:ADD-LAST('None'); DO v-LotEntry 1 TO NUM-ENTRIES(AcceptableValues,'|') slLot:ADD-LAST(ENTRY(...,AcceptableValues,'|')); else d-tsmsgbox MissingLotsTitle/MissingLots. BuildModifyFields: IF INDEX(ConfigOptionMaster.AcceptableValues, Modify.Lot)=0 THEN slLot='None' (or Fil-Lot=Modify.Lot, RS-Lot="Key" when v-LotKeyIn) ELSE slLot=Modify.Lot, Fil-Lot=""

# Impacted Systems

s-modify.w, ConfigOptionMaster (GlobalLotNumber, AcceptableValues), Modify.Lot, messages PrintLabels.Modify.MissingLotsTitle, PrintLabels.Modify.MissingLots.

# Traceability

- Lot choices come from AcceptableValues; 'None' is always an option unless already in the list.
- If config is missing or empty, the user is informed and the lot list is not built.
- On load, if current Modify.Lot is not in the list, it is treated as Key-In (if available) or defaulted to 'None'.

# Assertions

- AcceptableValues are split by '|' and each entry is added to slLot.
- When AcceptableValues is empty or ConfigOptionMaster record is missing, MissingLots (and MissingLotsTitle) are shown and no entries are added.
- BuildModifyFields ensures the displayed lot is either in the list or shown as Key-In when v-LotKeyIn and Modify.Lot is not in AcceptableValues.

# Related Information

- [[BR-Modify-003-Modify-screen-permissions]]
- [[BR-Modify-003-Modify-screen-permissions]]
- [[BR-Config-002-OperatorEdits-and-AcceptableValues]]
