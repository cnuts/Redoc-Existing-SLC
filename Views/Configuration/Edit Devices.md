---
Mode: Default
Authentication-Required: "true"
Authorization: Operator
---
# Purpose
Create, edit, delete devices.  [printers, scanners]
# Display Elements

- [[UI Chrome]]
- Device ID
	- Textbox, type text
	-  Validation:  Max, Min length
	- Required: true
	- default: none
- Description
	- Textbox, type text
	  - Validation: Max, Min length
	- Required: true
	- default: none
- Model
	- Drop Down List
	- [ ] Elements stored in [[Data Model]]  e.g [none, Wintronix, Datamax Prodigy] etc
	- Required: true
	- default: none
- IP Address
	- Textbox, text
	- [ ] Consider IP mask (safe to assume! we are not dealing with IP6)
	-  Validation:  Required, must match valid IP mask
	- Required: true
	- default: none
- COM Port
	- Textbox, integer
	- Validation:  must be 0-256 per limitation in Windows 11 (configurtion driven?)
	- Required: false
	- default: none
- Network ID
	- Textbox, text?
	- [ ] Not certain what this value is used for - needs description and use case
	- Required: false
	- default: none
- Baud
	- Textbox, integer
	- [ ] May wish to consider drop down list with defined values [300,1200,2400,9600,3600 etc]  [[Data Model]]
	- Required: true
	- default: none
- Input Buffer Size
	- Textbox, integer
	- [ ] May wish to consider a range of possible values preset in DDL
	- [ ] Validation: min/max
	- Required: true
	- default: none
- Parity
	-  Textbox, Single digit text
	-  [ ] may wish consider possible options in DDL [even, odd, mark, space, none]
	- Required: true
	- default: none
	- Default: none
- Output Buffer Size
	- Textbox, integer
	- [ ] Validation: min, max, are these predefined values?
	- Required: true
	- Default: none
- Data Bits
	- Textbox, integer
	- Validation: range of values 0 - 8?
	- Required: true
	- Default: none
- Timeout in MS
	- Textbox, integer
	- Validation: min/max
	- Required: true
	- default: none
- Stop Bits
	- Textbox, text value
	- Validation: consider a DDL of options [none, 0, 1.5, 2]  [[Data Model]]
	- Required: false
	- Default: none
- Generic Printer
	- Checkbox, Boolean
	- Required: false
	- Default: false
- DTR Enable
	- Checkbox, Boolean
	- Required: false
	- Default: false
- Attached
	- Checkbox, Boolean
	- Required: false
	- Default: false

- Quit
	- Button
	- Action: Return to [[0.Configuration Menu]]
# Redevelopment Notes

Follow the [[Patterns of Display]] for tabular data.



# Legacy Screens
![[Pasted image 20260306131851.png]]

