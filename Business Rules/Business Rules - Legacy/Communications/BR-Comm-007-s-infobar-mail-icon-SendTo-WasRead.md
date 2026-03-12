---
Risk-Level: P0
Business-Rule-Id: BR-Comm-007
Deprecated: false
---

# Description

The infobar (sobjects/s-infobar.w) displays a mail icon when there is at least one unread message addressed to the current scale. It finds the first CommHdr (primary DB) where CommHdr.SendTo = gICICTSID + STRING(gScaleID,'99') and CommHdr.WasRead = no, under no-lock no-error. If such a row is available, image-mail:visible is set to yes; otherwise image-mail:visible is set to no. Any error-status from the find is neutralized by ASSIGN v-Int = 0 NO-ERROR so that other smart objects are not affected. Thus the mail icon visibility is driven solely by the existence of an unread CommHdr for the current scale (SendTo = scale identifier).

# Source

- progress-SLC/sobjects/s-infobar.w: FIND first CommHdr WHERE SendTo = gICICTSID + string(gScaleID,'99') AND WasRead = no no-lock no-error. ASSIGN v-Int = 0 NO-ERROR. IF AVAIL CommHdr THEN image-mail:visible = yes. ELSE image-mail:visible = no.

# Impacted Systems

CommHdr, sobjects/s-infobar.w, image-mail widget, gICICTSID, gScaleID.

# Traceability

- SendTo format for scale-addressed messages is gICICTSID + two-digit scale ID; WasRead = no indicates unread.

# Assertions

- The check is a single FIND; any one unread message for the scale shows the icon; reading messages (WasRead = yes) or clearing them removes the icon when none remain.

# Related Information

- [[BR-Comm-001-CommDtl-CommHdr-ReadyToSend]]
- [[BR-Comm-004-messaging-host-outbound-WasRead-NO-send-then-WasRead-YES]]
