---
Mode: Default
Authentication-Required: "true"
Authorization: Operator
---
# Purpose

# Display Elements
- [[UI Chrome]]
- Table, existing operators
	- Data Source: [[Data Tables - Export-Import]]
		- Operator
		- First Name
		- Last Name
	- When a table row is selected, it is editable in the form.
		- Operator
			- Textbox, input
			- [ ] Validation:  max, min length, special characters
				- Required field
		- First Name
			- Textbox, input
			- [ ] Validation: max, min length, special characters
				- Required field
		- Last Name
		-  Textbox, input
			- [ ] Validation: max, min length, special characters
				- Required field
		- Password
		- - Password field, input
			- [ ] Validation: max, min length, special characters
			- [ ] [[Data Tables - Export-Import]]
			- [ ] [[Application Configuration]] ability to define the validation rules per configuration
				- Required field
		- Language
			- Drop down list, input, single selection
				- [[Data Tables - Export-Import]], [[Application Configuration]]  
					- [English, Spanish, French]
				- Default to English
		- Add
			- Button, input
				- Action: trigger validation
				- Action: if valid, save to database
				- Action refresh table display
				- State: disabled when?
		- Save
			- Button, input
				- Action: trigger validation
				- Action: if valid, update the user in the database
				- State: disabled when?
		- Delete
			- Button, input
				- Action: confirm deletion of selected user
				- Action: delete selected user
				- State: enabled only when user is selected from table
		- Reset
			- Button, input
				- Action: reset input form
		- Cancel
			- Button, input
				- Action: reset input form
				- State: disabled when?
		- Quit
			- Button, input
				- State: never disabled
				- Action:  Direct to [[0.Configuration Menu]]

- [ ] This entire form may be reconsidered for a modern UI.  Idea:  single table to display and inline crud operations. This would make the display portion smaller and accommodate touch screen keyboard better.  I think the "Quit" nomenclature  should only be used only to exit the application and replaced with "Back"

# Redevelopment Notes
Follow the [[Patterns of Display]] for Tabular Data

# Legacy Screens
![[Pasted image 20260306094350.png]]