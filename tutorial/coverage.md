# Coverage

Which chapter teaches each item of the compiler's catalog of forms, types, contexts and std packages (decision 104). A step uses only items named for its chapter or an earlier one. `internal/forms/cmd/doccheck` checks this file; `Item` is `<kind>:<id>`, and rows `phrase:<pattern>` may name std phrases. Every item is used by a step of its chapter, except `form:define-native` and `type:value`, which a chapter explains in prose only.

| Item | Shown as | Reference | Chapter |
| --- | --- | --- | --- |
| `form:header` | `# package main` | spec/06 §6.1 | 1 |
| `form:comment` | `# Say hello.` | spec/01 §1.3 | 1 |
| `form:block` | indented lines under `define main` | spec/01 §1.2 | 1 |
| `form:paren` | `(apples times price)` | spec/02 §2.3 | 2 |
| `form:member` | `n.Count` | spec/01 §1.4 | 4 |
| `form:list` | `{"Total: ", …}` | spec/04 §4.4 | 2 |
| `form:empty` | `{}` | spec/04 §4.4 | 8 |
| `form:continuation` | `otherwise` after a block | spec/02 §2.1 | 10 |
| `form:value-number` | `3`, `5.5` | spec/01 §1.5 | 2 |
| `form:value-duration` | `2s`, `150ms` | spec/01 §1.5 | 3 |
| `form:value-text` | `"Hello, Synod"` | spec/01 §1.6 | 1 |
| `form:value-color` | `#F4A340` | spec/01 §1.7 | 12 |
| `form:value-bool` | `true`, `false` | spec/04 §4.1 | 6 |
| `form:use` | `use std/lesson` | spec/06 §6.3 | 1 |
| `form:use-as` | `use … as base` | spec/06 §6.3 | 14 |
| `form:define` | `define Reply [m: msg]` | spec/03 §3.1 | 7 |
| `form:define-native` | `define … native "…"` | spec/03 §3.6 | 7 |
| `form:pattern-hole` | `[m: msg]` | spec/03 §3.1 | 7 |
| `form:pattern-receiver` | `define [n: Node] Reply …` | spec/03 §3.3 | 7 |
| `form:pattern-block` | `[b: block]` | spec/03 §3.5 | 10 |
| `form:run-block` | `run` block | spec/03 §3.5 | 10 |
| `form:main` | `define main` | spec/06 §6.5 | 1 |
| `form:start` | `define [n: Node] start` | spec/05 §5.3 | 4 |
| `form:has-node` | `Node has` | spec/05 §5.2 | 4 |
| `form:has-record` | `Ping has` | spec/04 §4.3 | 5 |
| `form:set-local` | `set total to …` | spec/02 §2.3 | 2 |
| `form:set-field` | `set n.Count to …` | spec/05 §5.2 | 4 |
| `form:set-item` | `set item k of … to …` | spec/04 §4.4 | 8 |
| `form:when-node` | `when [n: Node] (net.Received …)` | spec/05 §5.6 | 5 |
| `form:when-trace` | `when (sim.Changed …)` | spec/09 §9.1 | 12 |
| `form:when-typed` | `when [n: Node] (net.Received Ping from p)` | spec/05 §5.6 | 12 |
| `form:after` | `after 2s` | spec/07 §7.4 | 3 |
| `form:deliver` | `deliver … to …` | spec/07 §7.3 | 5 |
| `form:emit` | `emit … by n` | spec/05 §5.4 | 13 |
| `form:if` | `if (n.Round is 10)` | spec/02 §2.2 | 6 |
| `form:else` | `else` | spec/02 §2.3 | 6 |
| `form:each` | `each p in …` | spec/04 §4.4 | 8 |
| `form:each-pair` | `each k, v in …` | spec/04 §4.4 | 8 |
| `form:result` | `result …` | spec/03 §3.4 | 7 |
| `form:always` | `always (…)` | spec/05 §5.7 | 11 |
| `form:draw` | `draw … as …` | spec/09 §9.2 | 12 |
| `form:draw-for` | `draw … as … for …` | spec/09 §9.2 | 12 |
| `form:draw-block` | `draw … as … for …` with a block | spec/09 §9.2 | 12 |
| `form:attachment` | `name of v` | spec/09 §9.2 | 12 |
| `form:message` | `message Ping with …` | spec/04 §4.3 | 5 |
| `form:message-qualified` | `message pingpong.Start with …` | spec/04 §4.3 | 5 |
| `form:item` | `item 0 of nodes` | spec/04 §4.4 | 8 |
| `form:is` | `a is b` | spec/04 §4.5 | 6 |
| `form:is-not` | `a is not b` | spec/04 §4.5 | 6 |
| `form:is-in` | `k is in …` | spec/04 §4.4 | 8 |
| `form:is-not-in` | `k is not in …` | spec/04 §4.4 | 8 |
| `form:less-than` | `a less than b` | spec/04 §4.5 | 6 |
| `form:more-than` | `a more than b` | spec/04 §4.5 | 6 |
| `form:at-least` | `a at least b` | spec/04 §4.5 | 6 |
| `form:at-most` | `a at most b` | spec/04 §4.5 | 6 |
| `form:and` | `a and b` | spec/04 §4.5 | 6 |
| `form:or` | `a or b` | spec/04 §4.5 | 6 |
| `form:not` | `not a` | spec/04 §4.5 | 6 |
| `form:plus` | `a plus b` | spec/04 §4.5 | 2 |
| `form:minus` | `a minus b` | spec/04 §4.5 | 2 |
| `form:times` | `a times b` | spec/04 §4.5 | 2 |
| `form:divided-by` | `a divided by b` | spec/04 §4.5 | 2 |
| `type:text` | `"Hello"` | spec/04 §4.1 | 1 |
| `type:number` | `3` | spec/04 §4.1 | 2 |
| `type:bool` | `true` | spec/04 §4.1 | 6 |
| `type:time` | `2s` | spec/04 §4.1 | 3 |
| `type:list` | `list of text` | spec/04 §4.2 | 2 |
| `type:map` | `map of node to number` | spec/04 §4.2 | 8 |
| `type:node` | `[p: node]` | spec/04 §4.2 | 5 |
| `type:Node` | `[n: Node]` | spec/04 §4.2 | 4 |
| `type:msg` | `[m: msg]` | spec/04 §4.2 | 7 |
| `type:record` | `Ping` | spec/04 §4.2 | 5 |
| `type:shape` | `[s: shape]` | spec/04 §4.2 | 12 |
| `type:color` | `[c: color]` | spec/04 §4.2 | 12 |
| `type:field` | `[f: field]` | spec/04 §4.2 | 13 |
| `type:package` | `[p: package]` | spec/04 §4.2 | 4 |
| `type:block` | `[b: block]` | spec/04 §4.2 | 10 |
| `type:value` | `[v: value]` | spec/04 §4.2 | 8 |
| `context:node` | `define [n: Node] start` | spec/05 §5.4 | 4 |
| `context:main` | `define main` | spec/05 §5.4 | 1 |
| `context:check` | `always (…)` | spec/05 §5.4 | 11 |
| `context:look` | `when (sim.Changed …)` | spec/05 §5.4 | 12 |
| `package:std/disk` | `disk.Write` | std/README.md § std/disk | 10 |
| `package:std/lesson` | `lesson.Note` | std/README.md § std/lesson | 1 |
| `package:std/lists` | `lists.Append` | std/README.md § std/lists | 8 |
| `package:std/look` | `look.Card` | std/README.md § std/look | 12 |
| `package:std/look/basic` | the default look | std/README.md § std/look | 12 |
| `package:std/maps` | `maps.Size` | std/README.md § std/maps | 8 |
| `package:std/math` | `math.Floor` | std/README.md § std/math | 8 |
| `package:std/net` | `net.Send` | std/README.md § std/net | 5 |
| `package:std/random` | `random.Pick` | std/README.md § std/random | 9 |
| `package:std/sim` | `sim.StopAt` | std/README.md § std/sim | 3 |
| `package:std/text` | `text.Join` | std/README.md § std/text | 2 |
| `package:std/theme` | `theme.Accent` | std/README.md § std/theme | 12 |
| `package:std/time` | `time.Now` | std/README.md § std/time | 3 |
