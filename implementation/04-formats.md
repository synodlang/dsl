# File formats

All JSON is UTF-8 with keys in the order shown. Times are integers:
`t` in microseconds of simulated time, `at` in milliseconds of
presentation time. Every file starts with a `format` version; readers
reject unknown major versions. Within a major version keys are only added,
never removed or renamed, and readers ignore keys they do not know.

## synod.mod

```
module github.com/synodlang/examples
synod 0.3
require github.com/someone/raft v1.2.0
```

One directive per line: `module` (once), `synod` (language version,
once; newer than the compiler's is a `syntax` error), `require` (zero or
more, sorted; unsorted is `non-canonical`).

## synod.sum

`synod.sum` holds exactly the build list (every module of the minimal version selection, the main module excluded), one line per module, sorted by module path:

```
github.com/synodlang/examples v0.2.1 0f15d8d8f41f6fcaa68d4b2bb921da63884dfafc h1:<base64>
```

The line is `<module> <version> <commit> h1:<base64>`: the 40-hex commit of the tag, and the tree hash `h1:`, which is Go's `dirhash` Hash1. It is the base64 (standard alphabet, with padding) of the SHA-256 of the text made of one line `"<sha256 hex of the file>  <slash path>\n"` for every file of the module, paths relative to the module root and sorted by byte order; `.git` and any folder that holds its own `synod.mod` are not part of the module. An empty build list has no `synod.sum`: the file is removed.

The first download of a module with no line records it (trust on first use). Afterwards a store entry whose recomputed `h1:`, or a tag whose commit, differs from the line is exit 2 (decision 95).

Downloaded modules are not part of any format: `synod get` fills `vendor/` (which `.gitignore` excludes) with copies from the shared download store `~/.synod/cache/<module>@<version>/` (beside each entry, a file `<module>@<version>.commit` holds the tag's commit), downloading only what is missing and verifying every entry against `synod.sum`; `synod.sum` is what makes a fresh download reproducible.

## trace.jsonl

One JSON object per line. Line 1 is the header:

```json
{"format":"synod-trace/1","synod":"0.3","seed":7,"program":"github.com/synodlang/examples/paxos","packages":["github.com/synodlang/examples/paxos","std/net","std/sim"]}
```

`synod` is the language version. `seed` is the seed in force when `define main` returns (`spec/07` §7.7). `program` is the import path of the main file's folder; `packages` are the import paths of all other packages of the program, `std/` ones included, sorted by byte order.

Qualifiers in the trace (`paxos.Prepare`, `"paxos.Promised"`) are the package name; when two packages of the program share a name, the first in import-path order keeps `<name>` and the others get `<name>2`, `<name>3`. Numbers are written as ECMAScript `Number::toString` (`1e21` and above, and below `1e-6`, use an exponent without leading zeros; never `NaN`, `Infinity` or `-0`).

Positions (`pos`, and the path inside a `rule` text) are slash paths relative to the module, so a trace does not depend on the working directory: the main module's files from its root (`paxos/paxos.synod:40:5`), std files as `std/<package>/<file>`, vendored files as `<module path>@<version>/<path>`. Without a `synod.mod` the base name of the folder replaces the module root. The `pos` of `sim.Failed` is always in user code: an error raised inside std is placed at the call that entered std (decision 89).

Then one line per trace event:

```json
{"seq":12,"t":20431,"ev":"net.Sent","node":"n1","msg":{"id":5,"type":"paxos.Prepare","fields":{"number":"…"}},"from":"n1","to":"n3"}
{"seq":13,"t":31877,"ev":"net.Arrived","node":"n3","msg":{"id":5,"type":"paxos.Prepare","fields":{"number":"…"}},"from":"n1","to":"n3"}
{"seq":14,"t":31877,"ev":"sim.Changed","node":"n3","field":"paxos.Promised","old":0,"new":101}
{"seq":20,"t":52010,"ev":"sim.Emitted","node":"n2","record":{"type":"paxos.Learned","fields":{"Value":"x"}}}
{"seq":31,"t":90000,"ev":"sim.Broken","rule":"paxos/paxos.synod:125: always ((lists.Size ChosenValues) at most 1)"}
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
{"format":"synod-scene/1","duration":18450,"canvas":{"w":1040,"h":650},"theme":{"Paper":"#151A2E","Ink":"#EDE9E1","Line":"#3A4366","Quiet":"#8A90A8","Accent":"#F4A340","Good":"#5CC8B8","Warning":"#E8C35A","Critical":"#E86F5A"},"shapes":[
{"id":"s7a1f03c2b9d4","kind":"rect","props":{"w":96,"h":56,"radius":12},"keys":[{"at":0,"x":520,"y":300,"opacity":0,"fill":"Ink"},{"at":300,"opacity":1},{"at":5200,"fill":"Accent"}],"end":9000},
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
then `\n],"markers":[`, one marker per line, then, only when `camera` is
present, `\n],"camera":[` and one camera key per line, then `\n],"trace":[`,
one entry per line, and `\n]}\n`. An empty array is `[]` with no newline
inside. Line 1 holds `format`, `duration`, `canvas`, `theme` and, when it is
not empty, `style`, in that order.

- **Key order.** A shape: `id`, `kind`, `node`, `msg`, `props`, `keys`, then
  `end`. `node` and `msg` are omitted when not set (below). `end` is
  the presentation time the shape is deleted, and is omitted when it lives to
  the end of the scene. Inside a key: `at`, then `x`, `y`, `scale`,
  `opacity`, `rotation`, `fill`, `stroke`, `text`, `effect`, then (0.3) `curve`,
  `trail`, `dash`, `draw`, `halo`, `icon`, `count`, `role`, `face`. A marker: `at`,
  `kind`, `text`, then `choices` (only on `pause`, and only when not empty). The theme: Paper, Ink, Line, Quiet, Accent, Good, Warning,
  Critical, then any other used colors sorted by byte order (`Message1` to
  `Message4` appear only when a look used them, with their bare names).
- **First key.** The first key of every shape except a line sets `x`, `y` and
  `opacity`; a line's first key sets `opacity` and `stroke` (`"Line"`), no
  position. Every shape's first key also sets `scale` if not 1; `rotation` if not 0; `fill` and `stroke` when set; `text`
  for labels. Later keys set only the properties that change, strictly
  increasing in `at`. Between two keys the player interpolates numbers with the
  curve of their track (below), colors in OKLab, `icon` pointwise; `text`,
  `effect`, `curve`, `trail`, `dash`, `role` and `face` hold (step). `effect`
  is `"glow"`, `"indicate"`, `"flash"`, `"circumscribe"`, `"ripple"`,
  `"sweep"` or `"none"`; the builder writes `"none"` at the effect's start
  plus its duration scaled by pace. A group's members' `x` and `y` are
  relative to it.
- **Shapes.** `kind`: `circle` (`props` `{"r"}`), `rect`
  (`{"w","h","radius"}`), `label` (`{}`), `line` (`{"from","to"}`, shape ids;
  its ends follow those shapes, so it has no position keys), `group`
  (`{"items":[ids]}`), and, since 0.3, `icon` (`{"size":32}`; its first key
  sets `icon`, a name of the icon set) and `counter` (`{}`; its first key
  sets `count`). `id` is `s` and 12 lowercase hex digits
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

- **Style** (0.3). `style` is an object after `theme` on line 1 with the keys
  below in this order; a key is omitted when its value equals the one of
  `std/theme/default` (the default column) and the whole object is omitted
  when it would be empty. Readers use the default for a missing key.

| Key | Default | Meaning |
| --- | --- | --- |
| `sans`, `mono` | `"Inter"`, `"JetBrains Mono"` | family of face `sans` and `mono` (decision 118) |
| `titleSize`, `labelSize`, `valueSize`, `captionSize` | 28, 16, 15, 20 | text size of each role, in scene units |
| `titleWeight`, `labelWeight` | 600, 500 | weight of role `title`; of the other three roles |
| `tracking` | 0 | letter spacing in em |
| `stroke` | 2 | width of outlines and lines, in scene units |
| `glow` | 24 | blur of the glow effect and of `halo 1`, in scene units |
| `shadow` | 0 | alpha of the drop shadow, 0 to 1 |
| `vignette` | 0 | alpha of the vignette at the corners, 0 to 1 |
| `sketch` | 0 | amount of hand-drawn wobble, 0 to 1 |
| `drift` | 0 | amplitude of the idle camera drift, in scene units |
| `easing` | `"smooth"` | `"smooth"` or `"spring"` |

- **Camera** (0.3). `camera` is an array of keys `{"at","x","y","zoom"}`, one
  per line, strictly increasing in `at`, each with all three values; it is
  omitted when no `look.Focus` ran. Before the first key the camera is at
  the canvas centre `(w/2, h/2)`, `zoom` 1.
- **Key properties** (0.3). Each is written only when it changes, and its
  default holds before the first key:

| Property | Track | Default | Notes |
| --- | --- | --- | --- |
| `draw`, `halo`, `count` | number, smootherstep | `draw` 1, `halo` 0, `count` 0 | `draw` 0 to 1, `halo` 0 to 1, `count` a whole number |
| `icon` | morph, smootherstep | none | a name of the icon set (`icons.json`) |
| `curve`, `trail`, `dash`, `role`, `face` | step | `curve` 0, `trail` 0, `dash` `[]`, `role` `"label"`, `face` `"sans"` | `trail` in ms; `dash` is `[on, off]` in scene units; `role` is `"title"`, `"label"`, `"value"` or `"caption"`; `face` is `"sans"` or `"mono"` |

  A reader that does not know a property, a kind or a top-level key ignores it
  (format rule), so a 0.2 reader loads a 0.3 scene, draws an unknown `effect`
  as none and skips an `icon` or `counter` shape.
- **Icon set.** `implementation/icons.json` holds the 16 icons of `std/look`
  (`server`, `database`, `disk`, `user`, `client`, `clock`, `lock`, `mail`,
  `check`, `cross`, `warning`, `crown`, `bolt`, `cloud`, `key`, `eye`) as
  `{"format":"synod-icons/1","icons":{name:[contour, …]}}`. An icon has one to
  four contours; a contour is an open polyline of exactly 32 points `[x, y]`
  in a 1×1 box (origin top-left, y down), rounded to 4 decimals; a closed
  shape repeats its first point as its last. A reader draws an icon of `size`
  s by mapping `(x, y)` to `((x − 0.5)·s, (y − 0.5)·s)` around its center. The
  player ships a copy, checked against this file by a test.

### Drawing a scene (readers)

Readers draw a scene by these rules, so every conforming reader draws the
same picture. `at` is a presentation time in ms, a float for the player's
frames, clamped to `[0, duration]`.

- **Lifetime.** A shape is drawn for `firstKey.at ≤ at < end`; with no `end`,
  until the scene ends. A group draws nothing itself.
- **Tracks.** Each property is its own track of the keys that set it. Before
  the first key of a property, its default holds: `x` and `y` the canvas centre `w/2` and `h/2`,
  `scale` 1, `opacity` 1, `rotation` 0, `fill` and `stroke` none, `text`
  empty, `effect` `"none"`, and the defaults of the 0.3 properties above; a line's `stroke` defaults to `Line` and a label's
  `fill` to `Ink`. At and after the last key the last value holds; at a key's
  `at` its value holds.
- **Numbers.** Between two keys of a property, `v = a + (b − a)·F(u)`,
  `u = (at − a.at)/(b.at − a.at)`, with `F = E` for `x`, `y`, `scale`,
  `rotation` and the camera, and `F = S` for every other numeric property.
  `S(u) = u³(u(6u − 15) + 10)` (smootherstep). `E = S` when `style.easing` is
  `"smooth"`; when it is `"spring"`, `E(u) = 1 + (c + 1)(u − 1)³ + c(u − 1)²`
  with `c = 1.70158` (back-out; `E(0) = 0`, `E(1) = 1`, it overshoots). `opacity`
  is clamped to 0 to 1, `scale` to 0 or more, `draw` and `halo` to 0 to 1.
  `rotation` is in degrees, clockwise (y points down).
- **Colors.** Between two colors, convert the hex values (sRGB) to OKLab,
  interpolate L, a, b with `S(u)`, convert back, clamp and round to 8 bits.
  Between none and a color, or a color and none, the value steps at the later
  key. A name missing from `theme` draws as `Ink`. Colors always use `S`.
- **Steps.** `text`, `effect`, `curve`, `trail`, `dash`, `role` and `face` hold the value of the last key with
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
  transform. A label has no stroke. (0.3) Outlines and lines are `style.stroke` wide
  instead of 2 units. An `icon` of size `s` is stroked with the shape's
  `stroke` (else `Ink`), with round caps and joins, and has no fill. A `counter`
  is drawn as a label is.
- **Text.** (0.3) A label or counter has a role and a face. Its size is
  `style.<role>Size` scene units; its weight is `style.titleWeight` for role
  `title` and `style.labelWeight` for the others; its family is `style.sans`
  or `style.mono` by face; letters are `style.tracking` em apart (a reader
  without letter spacing draws 0). The size has no effect on hit testing or
  on the sizes the builder uses for layout (8 units per character, 18 high).
- **Lines.** A `line` joins the world centers of `from` and `to`, 2 units
  wide with round caps, in its own `stroke` at its own `opacity` times its
  group's. It is drawn only while both ends are drawn, and is never clipped
  to its ends.
- **Glow.** While `effect` is `"glow"`, the shape is drawn with a blur of
  `style.glow` scene units, colored by its `fill`, else its `stroke`, else
  `Ink`. An `effect` value the reader does not know draws as `"none"`, so new
  effects can be added within format 1.
- **Effects** (0.3). For `effect` values other than `"glow"` and `"none"`,
  with `p = (at − on)/(off − on)` where `on` is the key that set the value
  and `off` the next key of `effect`, the reader adds the drawing below on top
  of the shape. The effect color is the shape's `fill`, else its `stroke`,
  else `Ink`; `indicate` uses `Warning`. Every effect is multiplied by the
  shape's world opacity. The box of a shape is its `Place` size times its
  world scale (circle `2r` by `2r`, rect `w` by `h`, label 8 per character by
  18, icon `size` by `size`, counter 8 per column by 18); the box of a group
  is the union of the boxes of its drawn members, and the effect color of a
  group is that of its first member with a fill, else a stroke, else `Ink`.
  - `indicate`: the shape is scaled by `1 + 0.2·b` and its fill is mixed in
    OKLab toward `Warning` by `b`, with `b = 4p(1 − p)`.
  - `flash`: 12 rays at angles `k·30°`, each a line of 24 units, 2 units wide,
    from radius `R0 = half the box diagonal + 6` outward, at opacity `1 − p`.
  - `circumscribe`: an outline 8 units outside the box (a circle for a
    circle), 2 units wide, drawn on from its top center clockwise to the
    fraction `min(1, 2p)`, at opacity 1 while `p < 0.5` and `2(1 − p)` after.
  - `ripple`: a ring of radius `r0 + 60p` and opacity `1 − p`, where `r0` is
    half the longer side of the box.
  - `sweep`: a segment of 20% of the length, 4 units wide, in `Ink`, centered
    at the fraction `1.2p − 0.1` (clipped to 0 to 1) along a `line`, or along
    the outline of the box from its top center clockwise for any other shape.
- **Curve** (0.3). When the `x` and `y` keys A and B of a shape (the
  consecutive keys of both tracks, at the same `at`) bound a segment, the
  shape moves along a quadratic Bézier from A to B. Its control point is the
  midpoint of AB plus `c·|AB|` along the left normal `(dy, −dx)/|AB|`
  (y down), where `c` is the `curve` held at `A.at`, and its parameter is
  `F(u)` of the numbers rule. With `c = 0`, or when the keys of `x` and `y`
  do not coincide, the move is straight.
- **Trail** (0.3). With `trail = T > 0` ms held at `at`, 8 ghosts are drawn
  before the shape, oldest first: ghost `j` (1 to 8) is the shape as it was
  at `at − j·T/8` (position, scale, rotation, fill, stroke and opacity; no
  effect or halo), drawn at `0.5·opacity·(1 − j/9)`; a ghost before the
  shape's first key is skipped.
- **Draw** (0.3). With `draw = d`: a circle or rect draws its outline from its
  top center clockwise to the fraction `min(1, 2d)` of its perimeter (analytic:
  `2πr`, `2(w + h)` and corners by `radius`) and multiplies its fill alpha by
  `max(0, 2d − 1)`; a `line` draws from its `from` end toward its `to` end to
  the fraction `d`; an `icon` draws each contour to the fraction `d` of its
  own length; a label or counter shows its first `ceil(d·n)` code points
  (digits for a counter).
- **Dash** (0.3). `dash = [on, off]` strokes an outline or line with dashes of
  `on` and gaps of `off` scene units, starting at the start of its path;
  `on` 0 or an empty list is solid.
- **Halo** (0.3). With `halo = h` the shape is drawn with a shadow blur of
  `style.glow · h`, colored as the glow is. It combines with the glow effect
  by taking the larger blur. A glow or halo replaces the drop shadow.
- **Shadow** (0.3). A circle or rect with a fill, and an icon, is drawn with
  a shadow of offset (0, 6), blur 16 and color `rgba(0, 0, 0, style.shadow)`.
- **Sketch** (0.3). With `style.sketch = s > 0`, every outline (circle, rect,
  line, icon contour, effect stroke) is a polygon whose vertices lie every
  12 local units along its path (`max(4, ceil(length/12))` of them, evenly
  spaced; a closed outline does not repeat its first vertex). Vertex `i` is
  moved along the perpendicular of the chord from vertex `i − 1` to `i + 1` by
  `s·1.6·n(i)`, where `n(i)` is value noise at `i/3`: the lattice value at
  integer `k` is `2·m(k) − 1` with `m` the first output of mulberry32 started
  at `(seed + imul(k, 0x9E3779B1)) mod 2³²`, and values between lattice points
  are linear in `w = smoothstep(frac)`; on a closed outline the lattice index
  wraps modulo `ceil(count/3)`. The seed is `FNV-1a-32` of the shape's `id`
  (its 13 ASCII bytes), `seed2 = seed XOR 0x9E3779B9` for the second pass.
  An outline is drawn twice, first with `seed` at full alpha, then with
  `seed2` at alpha 0.6; a fill uses the first polygon. A line's ends overshoot
  by `4s` along its direction. The wobble depends on the shape and not on
  `at`, so it does not boil; labels and counters do not wobble.
- **Counter** (0.3). A counter draws `round(count)` as digit columns, the
  rightmost column k = 0 first. Column `k` is drawn while `k = 0` or
  `round(|count|) ≥ 10^k`; it is `0.6·size` wide, the row is centered on
  `(x, y)`, a minus sign precedes a negative number. With `a = |count|` and
  `q = a/10^k` the column shows glyph `floor(q) mod 10`, moved up by `r_k`
  of the line height (`1.2·size`), and glyph `(floor(q) + 1) mod 10` entering
  below it, clipped to the line. The roll `r_k` is an odometer's: the units
  column (`k = 0`) rolls with the fraction of the count, `r_0 = frac(a)`; a
  higher column waits until every lower column is on its 9 and then rolls
  with the units fraction, `r_k = max(0, (a mod 10^k) − (10^k − 1))`, which
  is `max(0, 10·frac(a/10) − 9)` for `k = 1`. So `r_k > 0` only while
  `a mod 10^k > 10^k − 1`, and a whole number has every `r_k = 0`: each
  glyph is drawn crisp, at its row, with no second glyph. As `count` moves
  between two whole numbers the columns roll together only where a carry
  happens (`19 → 20` turns the units and the tens column), and `count` that
  changes by less than one rolls the units column alone.
- **Icon morph** (0.3). Between two keys with `icon` names A and B, contour `i`
  of the drawn icon is `a_i + (b_i − a_i)·S(u)` pointwise, for each of the 32
  points; a contour that one icon lacks collapses to the center `(0.5, 0.5)`.
  An unknown name draws nothing.
- **Vignette** (0.3). After everything else, in canvas space (not moved by the
  camera) and clipped to the canvas, a radial gradient centered on the canvas,
  transparent up to `0.35·d` and `rgba(0, 0, 0, style.vignette)` from `0.75·d`
  outward, linear between, where `d` is the canvas diagonal.
- **Order.** Shapes are drawn in file order, later over earlier.
- **Unknown kinds.** A shape whose `kind` is unknown is skipped, with the
  lines and groups that refer only to it.
- **View.** The `canvas` is scaled by `min(width/w, height/h)` and centered
  in the reader's area (letterbox). The area is filled with `Paper`
  and the canvas rectangle clips what is drawn.
- **Camera and drift** (0.3). Shapes are drawn through the world view
  `translate(w/2, h/2) · scale(zoom) · translate(−cx', −cy')` (`(w, h)` the canvas) inside the
  letterbox, where `zoom` and `(cx, cy)` are the camera at `at` (its keys use
  `F = E`) and `cx' = cx + D·sin(2π·at/19000)`, `cy' = cy + 0.6·D·sin(2π·at/23000)`
  with `D = style.drift`. With no camera keys and `D = 0` the view is the
  identity.
- **Determinism** (0.3). Display lists are exact: the same scene and `at` give
  the same list of operations and numbers in every reader, because every
  function above is a polynomial, a fixed hash or a stated constant; `sin` is
  needed only for rotation, effects and drift. Pixels may differ slightly
  between engines (blur, gradient and font rasterization), so pixel tests use
  a tolerance and video exports compare by PSNR. The builder writes none of
  the per-frame details; it only writes keys, rounded as above.
- **Markers** are not drawn on the stage; a host shows them on its timeline.

### Hit testing (readers)

A reader that lets the viewer click the stage answers `hitTest(x, y)` by
these rules (decision 84), so every reader gives the same answer. It needs no
font metrics.

- The test runs on the frame last drawn. `(x, y)` is in CSS px of the reader's
  area; it is mapped through the inverse of the letterbox view, then of the
  camera view with drift, then of the world transform of each shape.
- **Candidates** are drawn `circle`, `rect`, `icon` and `counter` shapes (an
  icon is its square of side `size`, a counter its box of 8 units per column
  by 18), which hit like a shape with a fill. A candidate's `node` is
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

### Palette of a GIF export (hosts)

The one global palette of decision 88 is built from the scene's `theme` colors
(every name in the object, in OKLab). From 0.3 it also holds `Paper` with its
OKLab lightness `L` multiplied by 0.5 and by 0.25 whenever `style.vignette` or
`style.shadow` is above 0, so vignette and shadow tones have entries.

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

## overlay.json (`synod check --overlay`, `synod doc --overlay`)

```json
{"format":"synod-overlay/1","replace":{"gossip/gossip.synod":"/tmp/overlay/1.synod"}}
```

An editor uses it to check text that is not saved and to list the names such a file can write (decision 80), as Go's
`-overlay` does. `replace` maps files to files: each key is an existing
`.synod` file or `synod.mod`, absolute or relative to the working directory;
each value is a file holding the content to check in its place. The compiler
checks the key's file as if it held that content, and reports diagnostics and
tokens with the key's path and positions. Nothing is written. A new file has
to be saved before it is checked. Only `synod check` and `synod doc` take `--overlay`; a
missing or unreadable overlay file, a bad `format` or a key that is not an
existing `.synod` file or `synod.mod` is exit 2.

## synod-doc/1 (`synod doc --json`)

```json
{"format":"synod-doc/1","diagnostics":[],"scope":null,"packages":[
 {"path":"std/net","name":"net","qualifier":"net","title":"the simulated network","std":true,"activates":false,
  "phrases":[{"pattern":"net.Send [m: msg] from [a: node] to [b: node]",
    "items":[{"word":"net.Send"},{"hole":"m","type":"msg"},{"word":"from"},{"hole":"a","type":"node"},{"word":"to"},{"hole":"b","type":"node"}],
    "blocks":[],"receiver":null,"gives":"—","contexts":["node"],"kind":"phrase","native":true,"exported":true,
    "doc":"Send through the network from `a`, which must be the receiver",
    "def":{"file":"std/net/net.synod","line":3,"col":1}}],
  "records":[],"node":null}]}
```

Keys are written in this order: `format`, `diagnostics`, `scope`, `packages`.

- `diagnostics` are the diagnostic objects of `synod-check/1`; with any, `synod doc` exits 1.
- `packages` lists the documented packages. `path` is the import path (`std/net`, or the module path plus the folder); `qualifier` is how the reader writes the package (the alias under `use … as`); `title` is `""` when the package has none; `activates` is true when the package has observers or invariants, which importing it activates.
- A phrase has `pattern` (printed by the compiler, qualified as written outside the package), `items` (the pattern's items: `{"word"}` or `{"hole","type"}`), `blocks` (`[{"word":"otherwise","hole":"b"}]`, with `""` for an unnamed block hole), `receiver` (`{"name":"n","type":"Node"}` or null), `gives` (`—` for an action, else the Gives text of `std/README.md`), `contexts` (std only, else `null`), `kind` (`phrase`, `node-event` or `trace-event`), `native`, `exported`, `doc` (the doc comment, `spec/06` §6.4; `""` when none) and `def` (the declaration's position).
- A theme package (0.3) has, after `title`, the keys `"kind":"theme"` and `"values"`, and `phrases` `[]`: `values` lists its definitions in file order as `{"name","type","value","doc","def"}`, with `type` `color`, `number` or `text` and `value` the literal as JSON (a color as its hex text). A package that is not a theme has neither key. Readers ignore keys they do not know.
- A record is `{"name","doc","fields":[{"name","type","exported"}]}`; `node` is `{"doc","fields":[…]}`, or null when the package has no `Node has`.
- `scope` is null, or with `--scope <file>` (needs `--json`):

```json
"scope":{"file":"twopc/demo.synod","package":"main","imports":[{"path":"github.com/synodlang/examples/twopc","qualifier":"twopc"}]}
```

  `packages` then holds the file's own package (all its names, unexported ones included, with bare heads) and each import (exported names, heads spelled with that file's qualifier or alias), in import order. Readers ignore unknown keys.

## synod-explain/1 (`synod explain --json`)

```json
{"format":"synod-explain/1","codes":[
 {"code":"unknown-phrase","kind":"compile","summary":"No phrase matches the sentence by its shape.","text":"…",
  "command":"check","example":{"files":{"synod.mod":"module example.com/p\n","p/p.synod":"# package p\n…"},"program":""},
  "diagnostic":"p/p.synod:3:5: error[unknown-phrase]: …",
  "fix":{"note":"Parenthesize the condition.","files":{"p/p.synod":"…"},"deleted":[]}}]}
```

- `kind` is `compile` or `run`; `command` is `check` or `run`, and `program` is the folder `synod run` runs (`""` for `check`).
- `example.files` holds every file the example is checked or run with, its `synod.mod` included (decision 111).
- `diagnostic` is the short form lines the compiler printed when the document was made, never a stored text (decision 93).
- `fix.files` replace the example's files of the same path and `fix.deleted` removes files. Without a code, `codes` lists every code in the order of `spec/11`.
