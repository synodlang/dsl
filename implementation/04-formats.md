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
{"at":4100,"kind":"pause","text":"Who becomes leader next?"},
{"at":6000,"kind":"caption","text":"Raft never has two leaders in one term."},
{"at":9000,"kind":"broken","text":"always ((lists.Size ChosenValues) at most 1)"}
],"trace":[
{"seq":12,"at":1240}
]}
```

Line 1 ends with `"shapes":[`; each shape is one line, separated by `,\n`;
then `\n],"markers":[`, one marker per line, `\n],"trace":[`, one entry per
line, and `\n]}\n`. An empty array is `[]` with no newline inside.

- **Key order.** A shape: `id`, `kind`, `props`, `keys`, then `end`. `end` is
  the presentation time the shape is deleted, and is omitted when it lives to
  the end of the scene. Inside a key: `at`, then `x`, `y`, `scale`,
  `opacity`, `rotation`, `fill`, `stroke`, `text`, `effect`. A marker: `at`,
  `kind`, `text`. The theme: Paper, Ink, Line, Quiet, Accent, Good, Warning,
  Critical, then any other used colors sorted by byte order.
- **First key.** The first key of every shape sets `x`, `y` and `opacity`;
  `scale` if not 1; `rotation` if not 0; `fill` and `stroke` when set; `text`
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
- **Colors** are names, never raw hex: a `std/theme` name bare (`Accent`),
  another `<qualifier>.<Name>` (`raftlook.Hot`); every name used is in
  `theme`.
- **Numbers.** Numbers the builder computes are rounded to 0.01; the encoder
  fails on a non-finite number. Times are integer ms.
- **Markers.** `kind`: `pause` (a `lesson.Ask` question), `caption` (a
  `lesson.Note`), `broken` (the rule of `sim.Broken`), `failed` (a
  `sim.Failed` fact, `<pos>: error[<code>]: <message>`, or a look fault,
  `look <pos>: error[<code>]: <message>`). `caption` and `pause` are
  timeline-only. **Readers ignore a marker kind they do not know.**
- `trace` maps each trace `seq` to its presentation slot, so the player can
  show "simulated time" and jump to a fact.

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
  `\n`. A fix across files has text only. No tool applies an edit in M4;
  `synod fmt` applies only the canonical fixes.
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
- The WebAssembly API returns these documents as text, identical to the CLI's (`01-compiler.md`).
