---
Mode: Default
Authentication-Required: "false"
Authorization: none
---
# Purpose
# Display Elements
- Chrome
- Plant: Read-only, text 
- Shift: Text input
- Operator: text input
- Password: text input
- Login: button 
	- [ ] action, [[Authentication and Authorization]]
- Cancel: button -
	- [ ] follow on action
- [[Touch keyboard]]
	- [ ] when appear, when kid

# Validation Rules
Required: [Shift, Operator, Password]
Field Length and Content
- Shift:  Int (1)
- Operator: text - min 5, max 20
- Password: Password mask, min 5, max 20
	- [ ] Password Content Rules

- [ ] Define mode variants
- [ ] Magic spots
- [ ] Scan for other scales with the same ID

Action:
Check for [[login service]] based on setting, else authenticate locally
Action:
- [ ] Prior to login, the printer needs to check the network for other scales with the same scale ID.  I'm not sure exactly how this would happen.  A broadcast may not find other scales - is there a centralized way?  Perhaps the login service can be extended to identify hostname?

# Redevelopment Notes


![[20260305_155605.jpg]]