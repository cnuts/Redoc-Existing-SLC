---
Mode: Default
Authentication-Required: "true"
Authorization: Operator
Keep: "true"
Refactor: "true"
Deprecate: "false"
---
# Purpose
Edit items
- [ ] Define: what is an "item" and why do I want to edit it?
# Display Elements

- Item Code
- Lot
	- Drop down
	- [ ] Multiple select?
	- [ ] [[Data Model]]
	- Required: true
	- Default: "None"
- Price
	- textbox, floating point eg NNN.NN
	- Validation: max/min length value
	- Required: false
	- Default: none
- Customer
	- textbox, text
	- Validation: max/min length value
	- Required: true
	- Default: none
- Order Number
	- textbox, text
	- Validation: is this customer driven?  max/min length
	- Default: none
- Tare
	- textbox, text
	- Validation: min/max length
	- Default: none
- Kill Date Offset
	- An integer value defined as before or after a defined date.
	- This will likely store two values.  The offset integer 
	- eg:  -5 6/6/2026  or 3 5/5/2026
	- display localization
	- Validation:  rules as defined business rules
	- Default: today's date and a offset of 0
- Pack Date Offset
- An integer value defined as before or after a defined date.
	- This will likely store two values.  The offset integer 
	- eg:  -5 6/6/2026  or 3 5/5/2026
	- display localization
	- Validation:  rules as defined business rules
	- Default: today's date and a offset of 0
- Variable Offset
	- An integer value defined as before or after a defined date.
	- This will likely store two values.  The offset integer 
	- eg:  -5 6/6/2026  or 3 5/5/2026
	- display localization
	- Validation:  rules as defined business rules
	- Default: today's date and a offset of 0
# Redevelopment Notes
This seems best suited as editable tabular data.
[[Patterns of Display]]
# Legacy Screens
![[Pasted image 20260306154620.png]]