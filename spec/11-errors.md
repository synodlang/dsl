# 11. Errors

## 11.1 Format

Every diagnostic has a stable code, a location and one sentence, plus an
optional hint:

```
gossip/gossip.synod:14:3: error[unknown-phrase]: no phrase matches "net.Sned digest from n to peer"
  hint: did you mean "net.Send [m: msg] from [a: node] to [b: node]"?
```

`synod check --json` emits the same as JSON (`implementation/04-formats.md`).
Messages must be short and specific, because they are fed back to agents
verbatim.

## 11.2 Compile-time codes

| Code | Meaning |
| --- | --- |
| `syntax` | the structure grammar cannot parse the line; a comma outside list items, `message … with` settings and `each` names; a continuation line that follows no block; a number, text, comment or `use` written wrongly (`.5`, `+1`, `x-1`, a control character in text, `#todo`, a nested `use`, `()`) |
| `indent` | bad indentation (tabs, odd widths, unexpected indent) |
| `non-canonical` | parseable but not canonical (including CR LF line endings and a byte order mark); `synod fmt` fixes it |
| `bad-header` | missing `# package` header, or a name that is neither the folder name nor `main` |
| `unknown-phrase` | no phrase matches the sentence |
| `ambiguous-phrase` | more than one phrase matches |
| `duplicate-name` | a name defined twice in one package namespace (`spec/03` §3.2), including a second `start` |
| `shadowing` | a name hides another name in scope |
| `reserved-word` | a reserved word used as a name or head |
| `bad-pattern` | a define pattern breaks the rules in `spec/03` §3.1, including a head directly followed by a comparison operator |
| `bad-receiver` | a receiver whose type is not `Node` or `p.Node`; a `start` or `when` receiver that is not `Node`; a node-event `when` without a receiver, or a trace-event `when` with one; a receiver, `start`, node-event `when` or `Node has` in a main file |
| `bad-place` | `set` targets something other than a local, a node field, or an element of one (hole parameters, `when` bindings, `each` names and record fields cannot be set) |
| `type-mismatch` | an argument, operand, element or node type is not accepted (`spec/04` §4.7) |
| `ambiguous-empty` | cannot tell the type of `{}`: list or map, and its element types |
| `missing-result` | a value phrase has a path without `result` |
| `value-as-statement` / `action-as-value` | phrase kind used in the wrong place |
| `unexported` | using a lower-case name from another package |
| `unused-name` / `unused-import` | a local or `each` name is never used; an import is never referred to and its package has no observers or invariants |
| `unneeded-alias` | `use … as` without a collision |
| `unknown-import` / `import-cycle` | an import path names no library package (a missing folder, or one holding only a main file); imports form a cycle |
| `missing-main` / `extra-main` | a main file without `define main`, or `synod run` on a folder without a main file; a second main file in a folder, a second `define main`, or `define main` in a library file |
| `context` | a statement not allowed in its context (`spec/05` §5.4): e.g. `set` in an observer, a receiver phrase called from `define main`, `result` outside a phrase definition |
| `foreign-state` | node-context code reaches a node field or receiver phrase through anything but its receiver, or names a node other than the receiver after `from`, `on` or `by` |
| `color-literal` | a color literal outside a naming phrase |
| `native-outside-std` | `native` used outside `std/` |

## 11.3 Run-time errors

Recorded as `sim.Failed` (see `spec/07`): `missing-key`, `index-range`,
`disk-type` (a `disk.Read` value whose type differs from its
default), `division-by-zero`, `nobody`, `time-limit`.
