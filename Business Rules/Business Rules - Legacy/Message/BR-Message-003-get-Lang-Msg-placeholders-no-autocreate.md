---
Risk-Level: P0
Business-Rule-Id: BR-Message-003
Deprecated: false
---

# Description

The get-Lang-Msg function (lib/get-lang-msg.i) looks up Msg by LangCode and MsgId and returns the message text with placeholders replaced. It does not auto-create missing Msg rows; if no row is found it returns ip-MsgId. When a row is found, it replaces [%1], [%2], [%3], ... in Msg.Msg with the corresponding entries from ip-values, where ip-values is a pipe-separated list; replacement is in order (first value replaces [%1], second replaces [%2], etc.).

# Source

- progress-SLC/lib/get-lang-msg.i: FUNCTION get-Lang-Msg(ip-LangCode, ip-MsgId, ip-values); FIND msg by LangCode and MsgId no-lock no-error; if not avail msg then return ip-MsgId; op-msg = msg.Msg; DO v-inputCount = 1 to num-entries(ip-values,"|"): v-FromString = "[%" + trim(string(v-inputCount)) + "]", op-msg = replace(op-msg, v-FromString, entry(v-inputCount, ip-values, "|")); return op-msg

# Impacted Systems

Msg table, get-Lang-Msg (lib/get-lang-msg.i), any caller passing placeholder values (e.g. get-Lang-msg with string(gMaxBackDate) for ConfigMaxExceed).

# Traceability

- Placeholder format: [%1], [%2], ... up to number of pipe-separated entries in ip-values.
- No side effects: get-Lang-Msg never creates or updates Msg (unlike get-Lang-Lbl; see [[BR-Message-002-auto-create-missing-Msg-English-then-prefix]]).

# Assertions

- If FIND msg fails, return value is ip-MsgId unchanged.
- Replacement is sequential: entry 1 of ip-values → [%1], entry 2 → [%2], etc.
- Msg.Msg is not truncated by MsgLen in get-Lang-Msg; the full substituted string is returned.

# Related Information

- [[BR-Message-001-LangCode-MsgId-unique]]
- [[BR-Message-002-auto-create-missing-Msg-English-then-prefix]]
