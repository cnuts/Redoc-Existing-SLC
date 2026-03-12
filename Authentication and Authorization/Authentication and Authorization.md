

# Authentication

- [ ] User authentication
- [ ] API authentication - remotes
- [ ] There are more than a few configuration forms that do not need to be replicated if we go with RBAC - not to mention it is a much better idea.

# Authorization

RBAC
Map roles to permissions
Map users to roles.
Allow the creation of custom roles with their own bag of permissions.

Role Examples
	Device Administrator
	Operator
	Shift 1 Operator
	Shift 2 Operator
	Shift 3 Operator
	Label Administrator
	

# Business Rules
- [[BR-Auth-001 Username and Password]]
- [[BR-Auth-00002 Password storage must stored with SHA512 or greater and include a unique salt]]
- [[BR-Auth-00003- Password rules must be configurable.]]
