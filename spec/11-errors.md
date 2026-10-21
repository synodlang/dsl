# 11. Errors

## 11.1 Format

A diagnostic has a stable code, a location and one sentence, plus an
optional hint. It is written in two forms.

**Short form.** The form agents read, and what `synod check` prints when its
output is not a terminal:

```
gossip/gossip.synod:14:3: error[unknown-phrase]: no phrase matches "net.Sned digest from n to peer"
  hint: did you mean "net.Send [m: msg] from [a: node] to [b: node]"?
```

A diagnostic with no position (a folder without a main file, a run that
reaches the time limit) is `path: error[code]: message`, without `:line:col`.

`synod check --json` emits the same as JSON (`implementation/04-formats.md`).
Messages must be short and specific, because they are fed back to agents
verbatim.

**Rich form.** The form for people at a terminal (`synod check --style rich`):
the source line with markers under the exact span, labelled secondary spans,
notes, and the fix shown as a diff.

```
p/p.synod:13:5: error[unknown-phrase]: no phrase matches "if n.Role is "leader""
   |
13 |     if n.Role is "leader"
   |     ^^^^^^^^^^^^^^^^^^^^^ no phrase takes these items
   |        ------------------ 3 items here; "if" takes one condition
   |
help: parenthesize the condition
   |
13 |     if (n.Role is "leader")
   |        +                  +
```

The invariants of the rich form:

- Line 1 is exactly the first line of the short form. There is no `-->` line.
- The position `line:col` in line 1 is the start of the primary span, so the
  two forms cannot drift apart.
- The primary span is marked `^`, secondary spans `-`. A fix with edits shows
  the edited lines with `+` under inserted text, `~` under replaced text, or
  `-` and `+` rows when a whole line changes.
- No escape sequences unless color is on (`implementation/05-cli.md`). Output
  always ends lines with `\n`, on every system.
- A diagnostic without a position, or in a file the renderer cannot read, is
  its first line plus any notes and help, without a source line.

The hint drives the short form. Labels, help and notes drive the rich form. A
diagnostic that has a hint and no help shows the hint as `= help:` in the rich
form, so a producer can add structure one code at a time.

Text style of labels, help and notes: a label is at most about 50
characters, starts lower-case and has no final period; quoted words use
double quotes (`"if"`), never backticks; help is an imperative (`parenthesize
the condition`); a note is a statement. None repeats the message.

Run-time errors render the same way. `synod run` prints the `failed at
<time>: <pos>: error[<code>]: <message>` line first, unchanged, and under it
the source line of the failure.

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
| `unknown-import` / `import-cycle` | an import path names no library package (a missing folder, or one holding only a main file, an unknown `std/` package, a module that is not required or not in `vendor/`, a version that differs from the selected one, `github.com/synodlang/std/…`, a last segment that is not a word); imports form a cycle |
| `missing-main` / `extra-main` | a main file without `define main`, or `synod run` on a folder without a main file; a second main file in a folder, a second `define main`, or `define main` in a library file |
| `context` | a statement not allowed in its context (`spec/05` §5.4): e.g. `set` in an observer, a receiver phrase called from `define main`, a std phrase outside its catalog contexts, reading a lower-case field in look context, `emit … by` a node in main, an attachment outside look context, `result` outside a phrase definition or inside an `after` block or block argument, a non-declaration at file root, a `has` block line that is not `set <field> to <constant>` |
| `foreign-state` | node-context code reaches a node field or receiver phrase through anything but its receiver, or names a node other than the receiver after `from`, `on` or `by` |
| `color-literal` | a color literal outside a naming phrase |
| `native-outside-std` | `native` used outside `std/` |

## 11.3 Run-time errors

Recorded as `sim.Failed` (see `spec/07`) with the code, a message and the
source position (none for `time-limit`). The position in the trace stays the
start only (`synod-trace/1`); the span a failure underlines in the rich form
is not part of the trace. A diagnostic's position is a span (start and end)
internally, and `check.json` carries both.

| Code | Meaning |
| --- | --- |
| `missing-key` | `item k of m` with a key not in the map |
| `index-range` | a list index that is not a whole number in `0 ≤ i < size`, in `item` or `set item`; `random.Pick` of an empty list |
| `disk-type` | a `disk.Read` value whose type is not accepted as the type of its default (`spec/04` §4.7) |
| `division-by-zero` | `divided by` 0 or `0s`; `math.Remainder` by 0 |
| `nobody` | `sim.Nobody` where a node is needed: `net.Send` to it, `deliver` to it, `sim.Crash`, `sim.Restart`, `sim.Remove`, `net.Partition` or `net.DropNext` naming it, a node field of it |
| `time-limit` | the next event is later than 10 minutes of simulated time |
| `bad-argument` | an argument outside a phrase's domain: a negative `after` delay, `sim.Seed` not a whole number in 0..2^53, a negative `sim.StopAt`, `net.Latency` unless `0 ≤ a ≤ b`, `net.Loss` outside 0..1, a negative or fractional count, `random.Between` or `random.TimeBetween` with `b` below `a`, `lists.Take` or `lists.Range` with a non-whole argument |
| `number-range` | a non-finite number, or a time too large for 64 bits |
| `run-limit` | more than 100000 steps at one instant, call depth over 10000, or a list or text over 1000000 items |

### Look faults

A run-time error in look context (an observer, a timer of an observer, or a
reactive block) is a **look fault**. It uses the codes above but is never
recorded in the trace and is not `sim.Failed` (decision 68): the scene stops
at the fact being drawn, the partial scene is written with a `failed` marker
(`implementation/04-formats.md`), and `synod run` reports `look failed at
<time>: <pos>: error[<code>]: <message>` and exits 1
(`implementation/05-cli.md`). The cases:

| Code | Look cases |
| --- | --- |
| `missing-key` | `card of n` or `tag of k` with no shape attached under that name to that key: `no shape is attached to <key> under "<name>"`; `item k of m` with a missing key |
| `bad-argument` | `Tween` with a property other than the eight, or a value of the wrong kind or range (`"opacity"` outside 0..1, negative `"scale"`); a negative duration or `after` delay; `look.Flight` on a message not bound by a net event (`look.Flight needs a message bound by a net event`); `look.Layout` with another kind; `look.Effect` with another name; a second group for a shape; a `Place` cycle; a negative radius or size |
| `run-limit` | more than 100000 timer firings at one instant; call depth over 10000; more than 1000000 shapes; more than 10000000 keys |
| `index-range`, `division-by-zero`, `number-range` | as in the table above |
