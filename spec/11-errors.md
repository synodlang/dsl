# 11. Errors

## 11.1 Format

Every diagnostic has a stable code, a location and one sentence, plus an
optional hint:

```
gossip/gossip.synod:14:3: error[unknown-phrase]: no phrase matches "net.Sned digest to peer"
  hint: did you mean "net.Send [m: msg] to [n: node]"?
```

`synod check --json` emits the same as JSON (`implementation/04-formats.md`).
Messages must be short and specific, because they are fed back to agents
verbatim.

## 11.2 Compile-time codes

| Code | Meaning |
| --- | --- |
| `syntax` | the structure grammar cannot parse the line |
| `indent` | bad indentation (tabs, odd widths, unexpected indent) |
| `non-canonical` | parseable but not canonical; `synod fmt` fixes it |
| `bad-header` | missing `# package` header, or it does not match the folder |
| `unknown-phrase` | no phrase matches the sentence |
| `ambiguous-phrase` | more than one phrase matches |
| `duplicate-name` | a name defined twice in one package |
| `shadowing` | a name hides another name in scope |
| `reserved-word` | a reserved word used as a name or head |
| `bad-pattern` | a define pattern breaks the rules in `spec/03` |
| `type-mismatch` | an argument or operand has the wrong type |
| `ambiguous-empty` | cannot tell whether `{}` is a list or a map |
| `missing-result` | a value phrase has a path without `result` |
| `value-as-statement` / `action-as-value` | phrase kind used in the wrong place |
| `unexported` | using a lower-case name from another package |
| `unused-name` / `unused-import` | a local, an `each` name or an import is never used (`when` bindings, hole parameters, and imports of look or check packages are exempt) |
| `unneeded-alias` | `use … as` without a collision |
| `import-main` / `import-cycle` | illegal import |
| `missing-main` / `extra-main` | a `main` package without exactly one `main` |
| `context` | a statement not allowed in its context (`spec/05`), e.g. `set` in an observer |
| `foreign-state` | a node handler reads or writes another node's state |
| `color-literal` | a color literal outside a naming phrase |
| `native-outside-std` | `native` used outside `std/` |

## 11.3 Run-time errors

Recorded as `sim.Failed` (see `spec/07`): `missing-key`, `index-range`,
`type-mismatch`, `division-by-zero`, `nobody`, `foreign-state`,
`not-running` (reading a package's field on a node that does not run it),
`time-limit`.
