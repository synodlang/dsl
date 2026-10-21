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
| `syntax` | the structure grammar cannot parse the line; a comma outside list items, `message … with` settings and `each` names; a continuation line that follows no block; a number, text, comment or `use` written wrongly (`.5`, `+1`, `x-1`, a control character in text, `#todo`, a nested `use`, `()`); a duration that is not a whole number of microseconds, or a number out of 64-bit float range; a bad `synod.mod` line (an unknown directive, a repeated `module` or `synod`, a `synod` version newer than the compiler's) |
| `indent` | bad indentation (tabs, odd widths, unexpected indent) |
| `non-canonical` | parseable but not canonical (including CR LF line endings, a byte order mark, unsorted `require` lines, settings out of field order, a name that must start lower-case, and a package's own qualifier used inside it); `synod fmt` fixes it, when the package loads for the rows that need declarations (settings order, names, own qualifier) |
| `bad-header` | missing `# package` header, or a name that is neither the folder name nor `main`, starts upper-case or is a reserved word |
| `unknown-phrase` | no phrase matches the sentence by shape (types never decide, `spec/03` §3.3); no event phrase matches after `when`, or an event phrase stands elsewhere; a record has no such field; a `message … with` setting of more than two items |
| `ambiguous-phrase` | more than one phrase matches by shape |
| `duplicate-name` | a name defined twice in one package namespace (`spec/03` §3.2), including a second `start`, a record type named like a built-in type, two imports of one path or with one qualifier, a repeated hole name, and a field set twice in `message … with` |
| `shadowing` | a name hides another name in scope, including a record type of the package, and a node field or phrase head spelled like an import qualifier of the file |
| `reserved-word` | a reserved word used as a name or head |
| `bad-pattern` | a define or `when` pattern breaks the rules in `spec/03` §3.1, including a head directly followed by a comparison operator, a word after the head that starts upper-case, and an unknown type (`[x: Foo]`, `p.Node` naming no imported library package) |
| `bad-receiver` | a receiver whose type is not `Node` or `p.Node`; a `start` or `when` receiver that is not `Node`; a node-event `when` without a receiver, or a trace-event `when` with one; a receiver, `start`, node-event `when` or `Node has` in a main file |
| `bad-place` | `set` targets something other than a local, a node field, or an element of one (hole parameters, `when` bindings, `each` names and record fields cannot be set); a local from outside an `after` block or block argument set inside it |
| `type-mismatch` | an argument, operand, element or node type is not accepted (`spec/04` §4.7) |
| `ambiguous-empty` | cannot tell the type of `{}`: list or map, and its element types |
| `missing-result` | a value phrase has a path without `result` |
| `value-as-statement` / `action-as-value` | phrase kind used in the wrong place |
| `unexported` | using a lower-case name from another package |
| `unused-name` / `unused-import` | a local or `each` name is never read (setting it again is not a use); an import is never referred to and its package has no observers or invariants |
| `unneeded-alias` | `use … as` without a collision of last segments (two imports, or an import and the importing package's own name) |
| `unknown-import` / `import-cycle` | an import path names no library package (a missing folder, or one holding only a main file, an unknown `std/` package, a module that is not required or not in the cache, a version that differs from the selected one, `github.com/synodlang/std/…`, a last segment that is not a word); imports form a cycle |
| `missing-main` / `extra-main` | a main file without `define main`, or `synod run` on a folder without a main file; a second main file in a folder, a second `define main`, or `define main` in a library file |
| `context` | a statement not allowed in its context (`spec/05` §5.4): e.g. `set` in an observer, a receiver phrase called from `define main`, a std phrase outside its catalog contexts, reading a lower-case field in look context, `emit … by` a node in main, an attachment outside look context, `result` outside a phrase definition or inside an `after` block or block argument, a non-declaration at file root, a `has` block line that is not `set <field> to <constant>` |
| `foreign-state` | node-context code reaches a node field or receiver phrase through anything but its receiver, or names a node other than the receiver after `from`, `on` or `by` |
| `color-literal` | a color literal outside a naming phrase |
| `native-outside-std` | `native` used outside `std/` |

## 11.3 Run-time errors

Recorded as `sim.Failed` (see `spec/07`): `missing-key`, `index-range`,
`disk-type` (a `disk.Read` value whose type differs from its
default), `division-by-zero`, `nobody`, `time-limit`.
