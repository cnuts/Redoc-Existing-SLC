---
Mode: Default
---
# Purpose

A header bar across the top of the UI is always present with the following data.

Operator:  If logged in this field displays the operator ID.  If logged out is displays "Logout"

Application Version:  The version number of the SLC
- [ ] Assuming we still care about this.

Scale ID:  The ID of the scale.  Each scale has a unique ID value scoped to the plant
- [ ]  How is this formatted? What prevents duplicates on the same network.  Can this be setup in S9?

Count Up:  The number of unacknowledged messages sent to the queue 
- [ ] which queue and how is this calculated?

Date and Time:  [MM/DD date with HH:MMSS time] [[Application Configuration]]
- [ ] Is this configurable? where and what value?

The lightning bolt:  The lightning bolt across the top will flash when messages are sent and received.
- [ ] Are we sticking with this? Is the flashing universal for all messages?

- [ ] Why not a logout button in the header that returns to the user to the login page?
- [ ] Do we plan to let users exit the application?

# Redevelopment Notes

![[Screenshot 2026-03-06 084414.png]]
