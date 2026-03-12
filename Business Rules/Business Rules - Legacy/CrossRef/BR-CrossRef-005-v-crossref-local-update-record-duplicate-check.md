---
Risk-Level: P0
Business-Rule-Id: BR-CrossRef-005
Deprecated: false
---

# Description

In the CrossRef viewer (comm/v-crossref.w), when saving a new record (local-update-record), the procedure checks ADM-NEW-RECORD. If it is 'yes', before dispatching the standard update-record it finds another CrossRef (b-CrossRef) where Application equals the screen value of CrossRef.Application and ID equals the screen value of CrossRef.ID. If such a row exists, it shows the message "CrossRef.Application/ID Already Exists; Please Change CrossRef.Application / ID", applies 'entry' to CrossRef.Application, and returns without saving. Thus duplicate (Application, ID) is prevented on create from the UI.

# Source

- progress-SLC/comm/v-crossref.w: local-update-record. RUN get-attribute ("ADM-NEW-RECORD"). If vADM-new-record = 'yes', FIND FIRST b-CrossRef WHERE b-CrossRef.Application = CrossRef.Application:screen-value AND b-CrossRef.ID = CrossRef.ID:screen-value no-lock no-error. IF AVAIL b-CrossRef then run d-tsmsgbox "Create New CrossRef", "CrossRef.Application/ID Already Exists; Please Change CrossRef.Application / ID", apply 'entry' to CrossRef.Application, return. Then RUN dispatch 'update-record'.

# Impacted Systems

comm/v-crossref.w, CrossRef table, ADM add/update flow.

# Traceability

- Duplicate check is by (Application, ID) only; Descr is not part of the key.
- local-add-record only shows a prompt "Please set CrossRef.Application/ID and other values" before dispatching add-record; it does not validate uniqueness (uniqueness is enforced on save in local-update-record).

# Assertions

- Saving a new CrossRef with (Application, ID) that already exists is rejected with the stated message and no record is created.
- The check runs only when ADM-NEW-RECORD = 'yes' (i.e., new record from add or copy).

# Related Information

- [[BR-CrossRef-001-Application-ID-unique]]
