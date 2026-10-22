# File formats

All JSON is UTF-8 with keys in the order shown. Times are integers:
`t` in microseconds of simulated time, `at` in milliseconds of
presentation time. Every file starts with a `format` version; readers
reject unknown major versions. Within a major version keys are only added,
never removed or renamed, and readers ignore keys they do not know.

## synod.mod

```
module github.com/synodlang/examples
synod 0.2
require github.com/someone/raft v1.2.0
```

One directive per line: `module` (once), `synod` (language version,
once; newer than the compiler's is a `syntax` error), `require` (zero or
more, sorted; unsorted is `non-canonical`). `synod.sum` holds one line per
dependency: `<module> <version> <commit sha> <tree hash>`.

Downloaded modules are not part of any format: `synod get` fills `vendor/` (which `.gitignore` excludes) from the shared download store `~/.synod/cache/<module>@<version>/`, downloading only what is missing and verifying it against `synod.sum`; `synod.sum` is what makes a fresh download reproducible.

## trace.jsonl

One JSON object per line. Line 1 is the header:

```json
{"format":"synod-trace/1","synod":"0.2","seed":7,"program":"github.com/synodlang/examples/paxos","packages":["github.com/synodlang/examples/paxos","std/net","std/sim"]}
```

`synod` is the language version. `seed` is the seed in force when `define main` returns (`spec/07` §7.7). `program` is the import path of the main file's folder; `packages` are the import paths of all other packages of the program, `std/` ones included, sorted by byte order.

Qualifiers in the trace (`paxos.Prepare`, `"paxos.Promised"`) are the package name; when two packages of the program share a name, the first in import-path order keeps `<name>` and the others get `<name>2`, `<name>3`. Numbers are written as ECMAScript `Number::toString` (`1e21` and above, and below `1e-6`, use an exponent without leading zeros; never `NaN`, `Infinity` or `-0`).

Positions (`pos`, and the path inside a `rule` text) are slash paths relative to the module, so a trace does not depend on the working directory: the main module's files from its root (`paxos/paxos.synod:40:5`), std files as `std/<package>/<file>`, vendored files as `<module path>@<version>/<path>`. Without a `synod.mod` the base name of the folder replaces the module root.

Then one line per trace event:

```json
{"seq":12,"t":20431,"ev":"net.Sent","node":"n1","msg":{"id":5,"type":"paxos.Prepare","fields":{"number":"…"}},"from":"n1","to":"n3"}
{"seq":13,"t":31877,"ev":"net.Arrived","node":"n3","msg":{"id":5,"type":"paxos.Prepare","fields":{"number":"…"}},"from":"n1","to":"n3"}
{"seq":14,"t":31877,"ev":"sim.Changed","node":"n3","field":"paxos.Promised","old":0,"new":101}
{"seq":20,"t":52010,"ev":"sim.Emitted","node":"n2","record":{"type":"paxos.Learned","fields":{"Value":"x"}}}
{"seq":31,"t":90000,"ev":"sim.Broken","rule":"paxos/paxos.synod:109: always ((lists.Size ChosenValues) at most 1)"}
```

| `ev` | Extra keys |
| --- | --- |
| `sim.Started` | none |
| `sim.Ended` | `reason`: `"empty"`, `"stop"`; a failed run has no `sim.Ended` |
| `sim.Added`, `sim.Restarted` | `node`, `runs` (import paths), `state` (full exported state, `{ "pkg.Field": value }`) |
| `sim.Removed`, `sim.Crashed` | `node` |
| `sim.Changed` | `node`, `field`, `old`, `new` |
| `sim.Injected` | `node`, `record` |
| `sim.Fired` | `node` |
| `sim.Broken` | `rule` |
| `sim.Emitted` | `node` (`null` when emitted by `sim.Nobody` from main), `record` |
| `sim.Failed` | `code`, `message`, `pos` (`null` for `time-limit`) |
| `net.Sent`, `net.Arrived` | `node` (sender for `net.Sent`, receiver for `net.Arrived`), `msg`, `from`, `to` |
| `net.Dropped` | `node` (the receiver), `msg`, `from`, `to`, `reason` |
| `net.Partitioned` | `a`, `b` (lists of node names) |
| `net.Healed` | none |
| `disk.Written` | `node`, `key` |

Value encoding: text → JSON string; number → JSON number; bool → JSON
bool; time → `{"time":20000}` (µs); node → `{"node":"n3"}`; list → array;
map → `{"map":[[k, v], …]}` in key order; record → `{"type":…,"fields":{…}}`.
Only exported fields are written for records and state; a record's
unexported fields appear as `"…"` so looks cannot depend on them.

## scene.json

Read by the `player` library (the Mac app and the site both use it). Written
by `synod run` beside the trace, byte for byte as below: no spaces, numbers as
in the trace, one shape per line.

```
{"format":"synod-scene/1","duration":18450,"canvas":{"w":1600,"h":1000},"theme":{"Paper":"#151A2E","Ink":"#EDE9E1","Line":"#3A4366","Quiet":"#8A90A8","Accent":"#F4A340","Good":"#5CC8B8","Warning":"#E8C35A","Critical":"#E86F5A"},"shapes":[
{"id":"s7a1f03c2b9d4","kind":"rect","props":{"w":96,"h":56,"radius":12},"keys":[{"at":0,"x":800,"y":300,"opacity":0,"fill":"Ink"},{"at":300,"opacity":1},{"at":5200,"fill":"Accent"}],"end":9000},
{"id":"s0c3e91ab5d27","kind":"label","props":{},"keys":[{"at":0,"x":0,"y":40,"opacity":1,"fill":"Ink","text":"n1"}]}
],"markers":[
{"at":4100,"kind":"pause","text":"Who becomes leader next?","choices":["It times out and elects a new leader","It waits for the old leader"]},
{"at":6000,"kind":"caption","text":"Raft never has two leaders in one term."},
{"at":9000,"kind":"broken","text":"always ((lists.Size ChosenValues) at most 1)"}
],"trace":[
{"seq":12,"at":1240}
]}
```

Line 1 ends with `"shapes":[`; each shape is one line, separated by `,\n`;
then `\n],"markers":[`, one marker per line, `\n],"trace":[`, one entry per
line, and `\n]}\n`. An empty array is `[]` with no newline inside.

- **Key order.** A shape: `id`, `kind`, `node`, `msg`, `props`, `keys`, then
  `end`. `node` and `msg` are omitted when not set (below). `end` is
  the presentation time the shape is deleted, and is omitted when it lives to
  the end of the scene. Inside a key: `at`, then `x`, `y`, `scale`,
  `opacity`, `rotation`, `fill`, `stroke`, `text`, `effect`. A marker: `at`,
  `kind`, `text`, then `choices` (only on `pause`, and only when not empty). The theme: Paper, Ink, Line, Quiet, Accent, Good, Warning,
  Critical, then any other used colors sorted by byte order.
- **First key.** The first key of every shape except a line sets `x`, `y` and
  `opacity`; a line's first key sets `opacity` and `stroke` (`"Line"`), no
  position. Every shape's first key also sets `scale` if not 1; `rotation` if not 0; `fill` and `stroke` when set; `text`
  for labels. Later keys set only the properties that change, strictly
  increasing in `at`. Between two keys the player interpolates with the house
  curve (smootherstep); colors interpolate in OKLab; `text` and `effect` hold
  (step). `effect` is `"glow"` or `"none"`. A group's members' `x` and `y`
  are relative to it.
- **Shapes.** `kind`: `circle` (`props` `{"r"}`), `rect`
  (`{"w","h","radius"}`), `label` (`{}`), `line` (`{"from","to"}`, shape ids;
  its ends follow those shapes, so it has no position keys), `group`
  (`{"items":[ids]}`). `id` is `s` and 12 lowercase hex digits
  (`03-scene-builder.md`). A `line` or `group` may refer to the singleton
  `look.Center`, written as an empty group only then.
- **Node and message.** `node` is the name of the node (`"n3"`) a shape is
  attached to by `draw … as w for n`; with several node attachments the first
  counts. `msg` is the trace message id (`msg.id`, an integer) of the message
  a shape flies for in `look.Flight … for m`; the first flight counts. The
  scene builder writes both, so every look gets them, `std/look/basic`,
  `raftlook` and `ringlook` included. Readers do not draw with them; they are
  for hit testing. Label shapes may carry them too.
- **Colors** are names, never raw hex: a `std/theme` name bare (`Accent`),
  another `<qualifier>.<Name>` (`raftlook.Hot`); every name used is in
  `theme`.
- **Numbers.** Numbers the builder computes are rounded to 0.01; the encoder
  fails on a non-finite number. Times are integer ms.
- **Markers.** `kind`: `pause` (a `lesson.Ask` question; `choices` is the
  list of texts of `lesson.Ask … choosing`, 2 to 6 of them, and is omitted
  when the question has none), `caption` (a
  `lesson.Note`), `broken` (the rule of `sim.Broken`), `failed` (a
  `sim.Failed` fact, `<pos>: error[<code>]: <message>`, or a look fault,
  `look <pos>: error[<code>]: <message>`). `caption` and `pause` are
  timeline-only. **Readers ignore a marker kind they do not know.**
- `trace` maps each trace `seq` to its presentation slot, so a host can show
  simulated time (taking `t` from the trace by `seq`) and jump to a fact.

### Drawing a scene (readers)

Readers draw a scene by these rules, so every conforming reader draws the
same picture. `at` is a presentation time in ms, a float for the player's
frames, clamped to `[0, duration]`.

- **Lifetime.** A shape is drawn for `firstKey.at ≤ at < end`; with no `end`,
  until the scene ends. A group draws nothing itself.
- **Tracks.** Each property is its own track of the keys that set it. Before
  the first key of a property, its default holds: `x` and `y` 800 and 500,
  `scale` 1, `opacity` 1, `rotation` 0, `fill` and `stroke` none, `text`
  empty, `effect` `"none"`; a line's `stroke` defaults to `Line` and a label's
  `fill` to `Ink`. At and after the last key the last value holds; at a key's
  `at` its value holds.
- **Numbers.** Between two keys of a property, `v = a + (b − a)·S(u)`,
  `u = (at − a.at)/(b.at − a.at)`, `S(u) = u³(u(6u − 15) + 10)`. `opacity` is
  clamped to 0 to 1 and `scale` to 0 or more. `rotation` is in degrees,
  clockwise (y points down).
- **Colors.** Between two colors, convert the hex values (sRGB) to OKLab,
  interpolate L, a, b with `S(u)`, convert back, clamp and round to 8 bits.
  Between none and a color, or a color and none, the value steps at the later
  key. A name missing from `theme` draws as `Ink`.
- **Steps.** `text` and `effect` hold the value of the last key with
  `key.at ≤ at`. An instant change is written as a key one ms earlier that
  holds the old value, so no reader glides.
- **Groups.** Membership is fixed for the whole scene: the builder writes every
  member key, past ones too, relative to the group. A member's world
  transform is `parent ∘ translate(x, y) ∘ rotate(rotation) ∘ scale(scale)`,
  nested groups recursively; world opacity is the product of the opacities up
  the chain. A group's glow applies to its members. Before a group's first
  key its first-key values hold, so its members stay in place relative to it.
  A deleted group's members end with it (each carries its own `end`).
- **Geometry.** `(x, y)` is the center of every kind. A `circle` has radius
  `r`; a `rect` is `w` by `h` with corner radius `min(radius, w/2, h/2)`; a
  `label` is centered on `(x, y)` in both directions. The fill and the stroke
  are drawn only when set; the stroke is 2 scene units wide, in the shape's
  transform. A label has no stroke.
- **Lines.** A `line` joins the world centers of `from` and `to`, 2 units
  wide with round caps, in its own `stroke` at its own `opacity` times its
  group's. It is drawn only while both ends are drawn, and is never clipped
  to its ends.
- **Glow.** While `effect` is `"glow"`, the shape is drawn with a blur of 24
  scene units, colored by its `fill`, else its `stroke`, else `Ink`. An
  `effect` value other than `"glow"` draws as `"none"`, so new effects can be
  added within format 1.
- **Order.** Shapes are drawn in file order, later over earlier.
- **Unknown kinds.** A shape whose `kind` is unknown is skipped, with the
  lines and groups that refer only to it.
- **View.** The `canvas` is scaled by `min(width/w, height/h)` and centered
  in the reader's area (letterbox). The area is filled with `Paper`
  and the canvas rectangle clips what is drawn.
- **Markers** are not drawn on the stage; a host shows them on its timeline.

### Hit testing (readers)

A reader that lets the viewer click the stage answers `hitTest(x, y)` by
these rules (decision 84), so every reader gives the same answer. It needs no
font metrics.

- The test runs on the frame last drawn. `(x, y)` is in CSS px of the reader's
  area; it is mapped through the inverse of the letterbox view and the inverse
  world transform of each shape.
- **Candidates** are drawn `circle` and `rect` shapes. A candidate's `node` is
  its own, else the nearest enclosing group's; `msg` is resolved the same way
  and separately. A shape with neither never hits.
- **Slop** is 4 CSS px. A shape with a fill hits inside its geometry plus the
  slop. A shape with only a stroke hits within 1 scene unit (half the stroke
  width) plus the slop of its outline. A shape with neither fill nor stroke, a
  `label`, a `line`, and a shape whose world opacity is below 0.05 never hit.
- The topmost hit wins: the last in file order.
- The result has the shape's `id` and `kind`, and its resolved `node` and `msg`
  (each `null` when not set). A host may also ask for the box of a node: the
  union of the bounding boxes of its drawn circles and rects, without slop.

## check.json (`synod check --json`)

```json
{"format":"synod-check/1","ok":false,"diagnostics":[
  {"file":"gossip/gossip.synod","line":14,"col":3,"end":{"line":14,"col":34},"severity":"error","code":"unknown-phrase",
   "message":"no phrase matches \"net.Sned digest from n to peer\"","hint":"did you mean \"net.Send [m: msg] from [a: node] to [b: node]\"?",
   "label":"no phrase takes these items",
   "help":[{"text":"write \"net.Send\"","edits":[{"line":14,"col":3,"end":{"line":14,"col":11},"new":"net.Send"}]}]}
]}
```

- `line`, `col` and `end` form the primary span of the diagnostic. `line` and
  `col` are its start, as always; `end` (`{"line","col"}`) is the position
  just after it, so `end` equals the start for an empty span. `end` is always
  present. Columns count code points and `end` is exclusive.
- `hint` is the one-line hint of the short form, unchanged.
- `label` is the text under the primary span.
- `labels` are the secondary spans: `{"file","line","col","end","text"}`; the
  file may be another file.
- `help` lists fixes: `{"text", "edits"}`. An edit
  `{"line","col","end","new"}` replaces the span in the diagnostic's own file
  with `new`; an empty span inserts. Edits of one help do not overlap. An
  edit that adds whole lines inserts at column 1 with a `new` that ends in
  `\n`. A fix across files has text only. `synod fmt` applies only the canonical
  fixes; the Mac app applies a `help` edit only when the user picks it
  (decision 81).
- `notes` are statements.
- `label`, `labels`, `help` and `notes` are omitted when empty.

`synod check --json --tokens` adds `files`, sorted by path, each with its
tokens sorted by position, for editor highlighting and go-to-definition:

```json
{"format":"synod-check/1","ok":true,"diagnostics":[],"files":[{"file":"gossip/gossip.synod","tokens":[
 {"line":1,"col":1,"len":16,"role":"comment"},
 {"line":2,"col":1,"len":3,"role":"language"},
 {"line":7,"col":1,"len":6,"role":"type","def":{"file":"gossip/gossip.synod","line":7,"col":1}}]}]}
```

- One token per word, per part of a member (with its own column), per
  literal and per comment; punctuation is omitted. `len` is in code points.
- `role`: `language` (reserved words), `phrase` (heads and literal words of
  calls and patterns), `name` (locals, holes, bindings, packages, import
  paths), `field` (node fields, record fields, setting words), `type`
  (record types, `Node`, hole types), `value` (literals, `true`, `false`),
  `comment`.
- `def` points to the declaration; it is omitted for std and language words.
- Without `--tokens` the output has no `files` key.
- `files` lists every user file of the checked packages (never std or
  `vendor/`). A file that fails to lex or parse, and every other file of its
  package, is listed with lexical tokens only (decision 79): all its tokens,
  or those before its first lexical error. Their roles are `language` for
  reserved words; `value` for numbers, durations, texts, colors, `true` and
  `false`; `comment`; and `name` for every other word, part of a member and
  import path. They have no `def`. A parse error comes after a complete lex,
  so a file with only a parse error is highlighted in full.
- The WebAssembly API returns these documents as text, identical to the CLI's (`01-compiler.md`).

## overlay.json (`synod check --overlay`)

```json
{"format":"synod-overlay/1","replace":{"gossip/gossip.synod":"/tmp/overlay/1.synod"}}
```

An editor uses it to check text that is not saved (decision 80), as Go's
`-overlay` does. `replace` maps files to files: each key is an existing
`.synod` file or `synod.mod`, absolute or relative to the working directory;
each value is a file holding the content to check in its place. The compiler
checks the key's file as if it held that content, and reports diagnostics and
tokens with the key's path and positions. Nothing is written. A new file has
to be saved before it is checked. Only `synod check` takes `--overlay`; a
missing or unreadable overlay file, a bad `format` or a key that is not an
existing `.synod` file or `synod.mod` is exit 2.
