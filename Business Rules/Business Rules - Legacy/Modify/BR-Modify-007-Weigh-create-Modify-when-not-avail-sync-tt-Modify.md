---
Risk-Level: P0
Business-Rule-Id: BR-Modify-007
Deprecated: false
---

# Description

When opening the Modify screen from the weigh/print flow (weigh.w), the application ensures a Modify record exists for the current product (gProductCode). If no Modify row is found, one is created with only ProductCode (from tt-Product.ProductCode) and VarOffset (from tt-Product.VarOffset). The Modify screen (s-modify.w) is then run with the Modify buffer. After the user closes the screen, all tt-Modify rows are deleted and a single new tt-Modify row is created by buffer-copying the (possibly updated) Modify record, so the session uses the latest values.

# Source

- progress-SLC/print/weigh.w: FIND Modify WHERE modify.ProductCode EQ gProductCode; IF NOT AVAIL THEN DO TRANSACTION CREATE Modify ASSIGN Modify.ProductCode = tt-Product.ProductCode, Modify.VarOffset = tt-Product.VarOffset; RUN sobjects/s-modify.w (buffer modify); then 400-delete-trans FOR EACH tt-modify exclusive-lock DELETE tt-modify; 800-create-trans CREATE tt-modify, buffer-copy modify to tt-modify

# Impacted Systems

Modify table, tt-Modify, weigh.w, s-modify.w.

# Traceability

- Modify is keyed by ProductCode; creation on demand avoids missing record when user opens Modify from Print Label.
- Post-dialog sync: tt-Modify is replaced entirely by a single row copied from Modify so weigh/print logic uses the saved values.

# Assertions

- Only ProductCode and VarOffset are set when creating a new Modify record from weigh.w; other fields rely on database initial values or defaults.
- After s-modify.w returns, the in-memory tt-Modify is rebuilt from the persistent Modify buffer, not merged.

# Related Information

- [[BR-Modify-001-One-per-Product]]
- [[BR-Modify-004-Create-Modify-defaults-and-tt-Modify-refresh]]
