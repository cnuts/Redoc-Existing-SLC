---
Risk-Level: P0
Business-Rule-Id: BR-Labels-005
Deprecated: false
---

# Description

The copy-label-engine.p script deploys the Label Engine and Datamax Utilities to the local machine. It creates the directory c:\LabelEngine and c:\LabelEngine\Logs, then copies c:\cts\dev\LabelEngine\LabelEngine.exe to c:\LabelEngine\LabelEngine.exe. It then creates c:\Datamax and copies c:\cts\dev\Datamax\Utilities.exe to c:\Datamax\Utilities.exe. OS-ERROR is checked after the copy; if non-zero, a message is displayed (e.g. "You are not the owner of this file or directory" for error 1, "The file or directory you want to delete does not exist" for error 2). The Label Engine is a separate VB6 application (see Views/System/Label Engine.md) invoked when the menu item is selected; this script prepares its runtime location.

# Source

- progress-SLC/load/copy-label-engine.p: OS-CREATE-DIR 'c:\LabelEngine', 'c:\LabelEngine\Logs'; OS-COPY "c:\cts\dev\LabelEngine\LabelEngine.exe" "c:\LabelEngine\LabelEngine.exe"; OS-ERROR handling; OS-CREATE-DIR 'c:\Datamax'; OS-COPY "c:\cts\dev\Datamax\Utilities.exe" "c:\Datamax\Utilities.exe"
- Views/System/Label Engine.md: VB6 application by David, executed when menu item selected

# Impacted Systems

copy-label-engine.p, c:\LabelEngine, c:\LabelEngine\Logs, c:\Datamax, LabelEngine.exe, Utilities.exe, deployment/setup.

# Traceability

- Source paths are fixed (c:\cts\dev\LabelEngine, c:\cts\dev\Datamax); target is c:\LabelEngine and c:\Datamax.
- This is a deployment/setup rule, not a runtime label-printing rule.

# Assertions

- Directories are created only if they do not exist (OS-CREATE-DIR behavior).
- Copy overwrites existing files at the target; OS-ERROR reflects permission or path issues.

# Related Information

- [[BR-Labels-001-Set-Labels-path-resolution-lbl-1st-2nd-can]]
- Views/System/Label Engine.md
