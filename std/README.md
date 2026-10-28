# Standard library catalog

The standard library is the module `github.com/synodlang/std`, imported as
`std/<package>`. It is normative: implementations must provide exactly
these phrases with exactly these patterns. Each table lists the pattern,
what it gives back (— for action phrases), where it may be used, and
whether it is a native (N) or written in Synod (S).

In native patterns `value` stands for any type. The Gives column states
result types in terms of the holes ("same as `l`", "item type of `l`").
An item, key or default argument must be accepted as the list's item type, the map's key type or the field it is assigned to; `{}` arguments take their type from it.

Context abbreviations: **node**, **main**, **check**, **look** (`spec/05`).
"any" means every context.

Each code block headed `# package` is a whole file of std and equals that
file byte for byte (`time/every.synod`, `lesson/lesson.synod`,
`look/motion.synod`).

A change to this catalog is not done until `agent-skills` is regenerated
(`synod doc --catalog`, see `implementation/08-agent-skills.md`).

## std/sim — nodes, runs, faults

| Pattern | Gives | Context | Kind | Meaning |
| --- | --- | --- | --- | --- |
| `Seed [n: number]` | — | main | N | Sets the seed of the run's PRNG; `n` is a whole number from 0 to 2^53, otherwise `bad-argument`. The `--seed` flag, when given, wins over `sim.Seed`; without either the seed is 1. The trace header records the seed in force when `define main` returns. |
| `StopAt [t: time]` | — | main | N | End the run at simulated time `t` |
| `Spawn [p: package]` | `p.Node` | main | N | Create and start one node |
| `Cluster of [count: number] running [p: package]` | `list of p.Node` | main | N | Create and start `count` nodes |
| `Crash [n: node]` | — | main | N | Stop a node (`spec/07`) |
| `Restart [n: node]` | — | main | N | Restart a crashed node with fresh state |
| `Remove [n: node]` | — | main | N | Remove a node for good |
| `Id [n: node]` | number | any | N | 1, 2, 3… in creation order |
| `Name [n: node]` | text | any | N | `"n1"`, `"n2"`, … |
| `Nobody` | node | any | N | The "no node" value |
| `Nodes` | `list of node` | main, check, look | N | All nodes not removed, in creation order |
| `NodesRunning [p: package]` | `list of p.Node` | main, check, look | N | Nodes running `p` |
| `IsUp [n: node]` | bool | main, check, look | N | Not crashed and not removed |
| `StateRows [n: node]` | `list of text` | look | N | One line per exported field of `n`: name and value as text |

**Node event:** `Delivered [r: msg]`, a record injected with `deliver` (used as
`when [n: Node] (sim.Delivered Open)`).

**Trace events:** `Started`, `Ended`, `Added [n: node]`,
`Removed [n: node]`, `Crashed [n: node]`, `Restarted [n: node]`,
`Changed [f: field] of [n: node] from [old: value] to [new: value]`,
`Injected [r: msg] into [n: node]`, `Fired on [n: node]`,
`Broken [rule: text]`, `Emitted [e: msg] by [n: node]`,
`Failed [error: text]` (`spec/08`).

## std/net — the simulated network

| Pattern | Gives | Context | Kind | Meaning |
| --- | --- | --- | --- | --- |
| `Send [m: msg] from [a: node] to [b: node]` | — | node | N | Send through the network from `a`, which must be the receiver |
| `Latency between [a: time] and [b: time]` | — | main | N | Uniform delay per message (default 10ms–30ms) |
| `Loss [p: number]` | — | main | N | Probability each message is lost (default 0) |
| `Partition [a: list of node] from [b: list of node]` | — | main | N | Cut every link between the two groups |
| `Heal` | — | main | N | Remove all partitions |
| `DropNext [type: text] from [a: node] to [b: node]` | — | main | N | Drop the next message of that record type (unqualified name) on that link |
| `Id [m: msg]` | number | look | N | The message id of the net fact that bound `m` |
| `Type [m: msg]` | text | look | N | The unqualified record type name of `m` |

**Node event:** `Received [m: msg] from [a: node]` (used as
`when [n: Node] (net.Received m from a)`).

**Trace events:** `Sent [m: msg] from [a: node] to [b: node]`,
`Arrived [m: msg] from [a: node] to [b: node]`,
`Dropped [m: msg] from [a: node] to [b: node] because [r: text]`,
`Partitioned [a: list of node] from [b: list of node]`, `Healed`.

## std/time

| Pattern | Gives | Context | Kind | Meaning |
| --- | --- | --- | --- | --- |
| `Now` | time | node, main, check | N | Simulated time since the run started |
| `Every [d: time] [b: block]` | — | node, main | S | Run `b` now and every `d` |

```synod
# package time
# Run `b` now and every `d`
define Every [d: time] [b: block]
  b
  after d
    Every d
      b
```

## std/random — deterministic randomness

| Pattern | Gives | Context | Kind | Meaning |
| --- | --- | --- | --- | --- |
| `Pick [l: list of value]` | item type of `l` | node, main | N | A uniformly chosen item; an `index-range` run-time error on an empty list |
| `Between [a: number] and [b: number]` | number | node, main | N | Uniform in [a, b) |
| `TimeBetween [a: time] and [b: time]` | time | node, main | N | Uniform in [a, b), whole microseconds |
| `Chance [p: number]` | bool | node, main | N | True with probability `p` |

## std/lists

| Pattern | Gives | Context | Kind | Meaning |
| --- | --- | --- | --- | --- |
| `Size [l: list of value]` | number | any | N | Number of items |
| `Append [x: value] to [l: list of value]` | same as `l` | any | N | A copy of `l` with `x` added at the end |
| `Extend [a: list of value] with [b: list of value]` | same as `a` | any | N | A copy of `a` followed by every item of `b` |
| `Without [x: value] from [l: list of value]` | same as `l` | any | N | A copy without any item equal to `x` |
| `Take [n: number] from [l: list of value]` | same as `l` | any | N | The first `n` items (fewer if the list is shorter) |
| `Range [a: number] to [b: number]` | `list of number` | any | N | Whole numbers from `a` to `b`, inclusive; empty when `a` is more than `b` |
| `Sort [l: list of value]` | same as `l` | any | N | Ascending; items must all be numbers, all times or all text |

## std/maps

| Pattern | Gives | Context | Kind | Meaning |
| --- | --- | --- | --- | --- |
| `Size [m: map of value to value]` | number | any | N | Number of keys |
| `Keys [m: map of value to value]` | list of the key type of `m` | any | N | Keys in ascending order |
| `Without [k: value] from [m: map of value to value]` | same as `m` | any | N | A copy without key `k` |
| `KeysByValue [m: map of value to value]` | list of the key type of `m` | any | N | Keys ordered by ascending value, ties by key |

## std/math

| Pattern | Gives | Context | Kind | Meaning |
| --- | --- | --- | --- | --- |
| `Floor [x: number]` | number | any | N | Round down |
| `Ceil [x: number]` | number | any | N | Round up |
| `Min of [a: number] and [b: number]` | number | any | N | Smaller |
| `Max of [a: number] and [b: number]` | number | any | N | Larger |
| `Remainder of [a: number] by [b: number]` | number | any | N | `a − b × floor(a / b)` |
| `Hash [t: text]` | number | any | N | A fixed number in [0, 1) from the text: FNV-1a 64-bit over its UTF-8 bytes, mixed with the MurmurHash3 64-bit finalizer, then `(h >> 11) / 2^53`. Not random: it never touches the PRNG. |

## std/text

| Pattern | Gives | Context | Kind | Meaning |
| --- | --- | --- | --- | --- |
| `From [v: value]` | text | any | N | Text form: numbers without trailing zeros, durations canonical, nodes by name, `true`/`false` |
| `Join [l: list of text]` | text | any | N | Concatenate a list of texts |
| `Size [t: text]` | number | any | N | Number of characters |

## std/disk — storage that survives crashes

| Pattern | Gives | Context | Kind | Meaning |
| --- | --- | --- | --- | --- |
| `Write [v: value] at [k: text] on [n: node]` | — | node | N | Store `v` durably on `n`, which must be the receiver (instantly; records `disk.Written`) |
| `Read [k: text] on [n: node] or [d: value]` | same as `d` | node | N | The value stored on `n`, or `d` when nothing is stored; the result has the type of `d` (`spec/04` §4.4) |

**Trace event:** `Written [k: text] on [n: node]`.

## std/lesson — Predict mode and captions

| Pattern | Gives | Context | Kind | Meaning |
| --- | --- | --- | --- | --- |
| `Ask [q: text]` | — | main | S | Emit `Asked` with no choices; the player pauses and asks the viewer to predict |
| `Ask [q: text] choosing [c: list of text]` | — | main | S | Emit `Asked` with the choices `c`: 2 to 6 distinct, non-empty texts, else `bad-argument`; the player pauses and offers them |
| `Note [t: text]` | — | main | S | Emit `Noted`; the player shows a caption |
| `choices [c: list of text]` | same as `c` | main | N | Unexported: check `c` for `Ask … choosing` (used by it) |

```synod
# package lesson
use std/sim

Asked has
  set Question to ""
  set Choices to {}

Noted has
  set Text to ""

# Emit `Asked` with no choices; the player pauses and asks the viewer to predict
define Ask [q: text]
  emit (message Asked with Question q) by sim.Nobody

# Emit `Asked` with the choices `c`: 2 to 6 distinct, non-empty texts, else `bad-argument`; the player pauses and offers them
define Ask [q: text] choosing [c: list of text]
  emit (message Asked with Question q, Choices (choices c)) by sim.Nobody

# Emit `Noted`; the player shows a caption
define Note [t: text]
  emit (message Noted with Text t) by sim.Nobody

# Unexported: check `c` for `Ask … choosing` (used by it)
define choices [c: list of text] native "lesson.choices"
```

`choices` is not exported (a native, like `look.delete`). It returns `c` after
checking it, and fails with `bad-argument` (`lesson.Ask … choosing needs 2 to
6 different, non-empty choices`) unless `c` has 2 to 6 distinct, non-empty
texts, so `choosing {}` is never a second spelling of `Ask [q: text]`.

## std/theme — the program's theme values

A theme is a package whose files start `# theme <word>` (`spec/06` §6.7). The
program's theme is the theme its main file imports, else `std/theme/default`.
`std/theme` is the interface to it: each of these 34 names is a one-word
look-context native whose result is the value the program's theme gives, read
in a look as `theme.<Name>` (for example `theme.Paper`). A theme exports no
phrase, so `theme.<Name>` is the only spelling. The names are look context
only, so a theme never changes a run. Every theme defines each name exactly
once, with a value of the name's type inside the domain its Meaning gives
(`bad-theme`). Colors keep their bare names in the scene `theme`
(`spec/09` §9.5). `Sans` and `Mono` name only the four families of the font
set (decision 118).

| Pattern | Gives | Context | Kind | Meaning |
| --- | --- | --- | --- | --- |
| `Paper` | color | look | N | Scene ground color |
| `Ink` | color | look | N | Node cards and primary text color |
| `Line` | color | look | N | Links and borders color |
| `Quiet` | color | look | N | Secondary text and idle nodes color |
| `Accent` | color | look | N | Messages color: the one thing to look at |
| `Good` | color | look | N | Success and committed color |
| `Warning` | color | look | N | Candidate and pending color |
| `Critical` | color | look | N | Broken invariant and crashed color |
| `Message1` | color | look | N | First of four message colors, picked by `look.MessageColor` |
| `Message2` | color | look | N | Second of four message colors |
| `Message3` | color | look | N | Third of four message colors |
| `Message4` | color | look | N | Fourth of four message colors |
| `Vignette` | number | look | N | Darkening of the scene edges, 0 to 1 |
| `Sketch` | number | look | N | Hand-drawn wobble of outlines, 0 to 1 |
| `Glow` | number | look | N | Glow blur in units, 0 to 48 |
| `Shadow` | number | look | N | Shadow strength of filled shapes, 0 to 1 |
| `StrokeWidth` | number | look | N | Outline width in units, 0.5 to 6 |
| `CornerRadius` | number | look | N | Corner radius of rects in units, 0 to 40 |
| `NodeShape` | text | look | N | Shape of a node card: `"card"` or `"circle"` |
| `NodeFill` | text | look | N | Fill of a node card: `"solid"` or `"outline"` |
| `LinkOpacity` | number | look | N | Opacity of links, 0 to 1 |
| `Easing` | text | look | N | Curve of position, scale, rotation and camera moves: `"smooth"` or `"spring"` |
| `Drift` | number | look | N | Slow camera drift in units, 0 to 24 |
| `Sans` | text | look | N | Family of the sans face: `"Inter"`, `"JetBrains Mono"`, `"Source Serif 4"` or `"Shantell Sans"` |
| `Mono` | text | look | N | Family of the mono face: `"Inter"`, `"JetBrains Mono"`, `"Source Serif 4"` or `"Shantell Sans"` |
| `TitleSize` | number | look | N | Size of the title role in units, 12 to 64 |
| `LabelSize` | number | look | N | Size of the label role in units, 10 to 40 |
| `ValueSize` | number | look | N | Size of the value role in units, 10 to 40 |
| `CaptionSize` | number | look | N | Size of the caption role in units, 12 to 48 |
| `TitleWeight` | number | look | N | Weight of the title role: 400, 500, 600 or 700 |
| `LabelWeight` | number | look | N | Weight of the label, value and caption roles: 400, 500, 600 or 700 |
| `Tracking` | number | look | N | Letter spacing in em, -0.05 to 0.2 |
| `Margin` | number | look | N | Inset of `look.Frame` in units, 0 to 100 |
| `Fill` | number | look | N | Scale of layout radii and spacing, 0.5 to 1.5 |

## std/theme/default — the default theme values

`std/theme/default` is the theme of a program whose main file imports none, and
the template to copy for a new theme: its files start `# theme default`, and a
copy keeps all 34 names and changes values. A main file never imports it
(`non-canonical`, and `synod fmt` removes the `use`). Its values (the Synod
brand for the colors) are:

| Name | Value | Use |
| --- | --- | --- |
| `Paper` | `#151A2E` | scene ground |
| `Ink` | `#EDE9E1` | node cards, primary text |
| `Line` | `#3A4366` | links, borders |
| `Quiet` | `#8A90A8` | secondary text, idle nodes |
| `Accent` | `#F4A340` | messages, the one thing to look at |
| `Good` | `#5CC8B8` | success, committed |
| `Warning` | `#E8C35A` | candidate, pending |
| `Critical` | `#E86F5A` | broken invariant, crashed |
| `Message1` | `#F4A340` | message color 1 |
| `Message2` | `#7FB2F0` | message color 2 |
| `Message3` | `#C79BEA` | message color 3 |
| `Message4` | `#F08FB0` | message color 4 |
| `Vignette` | `0` | edges not darkened |
| `Sketch` | `0` | outlines not wobbled |
| `Glow` | `24` | glow blur |
| `Shadow` | `0` | no shadow |
| `StrokeWidth` | `2` | outline width |
| `CornerRadius` | `12` | rect corners |
| `NodeShape` | `"card"` | node cards are rects |
| `NodeFill` | `"solid"` | node cards are filled |
| `LinkOpacity` | `1` | links are opaque |
| `Easing` | `"smooth"` | smootherstep moves |
| `Drift` | `0` | no drift |
| `Sans` | `"Inter"` | default sans face |
| `Mono` | `"JetBrains Mono"` | default mono face |
| `TitleSize` | `28` | title role |
| `LabelSize` | `16` | label role |
| `ValueSize` | `15` | value role |
| `CaptionSize` | `20` | caption role |
| `TitleWeight` | `600` | title role |
| `LabelWeight` | `500` | label, value, caption roles |
| `Tracking` | `0` | no letter spacing |
| `Margin` | `4` | `look.Frame` inset |
| `Fill` | `1` | layout scale |

A name is one definition whose block is one `result` line, as in this
beginning of the file:

```
# theme default
# scene ground
define Paper
  result #151A2E

# node cards, primary text
define Ink
  result #EDE9E1
```

## std/look — shapes, motion, layout (look context only)

| Pattern | Gives | Kind | Meaning |
| --- | --- | --- | --- |
| `Circle [r: number]` | shape | N | A circle of radius `r` points |
| `Rect [w: number] by [h: number]` | shape | N | A rectangle with the corner radius `theme.CornerRadius` |
| `Label [t: text]` | shape | N | Text |
| `Line from [a: shape] to [b: shape]` | shape | N | A line that follows both shapes, at opacity `theme.LinkOpacity` |
| `Group [l: list of shape]` | shape | N | Shapes moved and faded together |
| `Card for [n: node]` | shape | S | The standard node card: a box and a name label, shaped and filled by `theme.NodeShape` and `theme.NodeFill` |
| `Fill [s: shape] with [c: color]` | — | N | Fill color (morphs) |
| `Stroke [s: shape] with [c: color]` | — | N | Outline color (morphs) |
| `Tween [p: text] of [s: shape] to [v: value] over [d: time]` | — | N | The only motion primitive (`spec/09`); of `"x"` or `"y"` it also takes a shape: that shape's position |
| `Effect [e: text] on [s: shape]` | — | N | `"glow"` (600ms), `"indicate"` (600ms), `"flash"` (500ms), `"circumscribe"` (900ms), `"ripple"` (700ms) or `"sweep"` (800ms), scaled by pace |
| `Layout [kind: text]` | — | N | `"ring"`, `"row"` or `"grid"` for node cards, spaced by `theme.Fill` |
| `Place [s: shape] above [t: shape]` | — | N | Position relative to another shape; likewise `below`, `beside` |
| `Place [s: shape] below [t: shape]` | — | N | Position relative to another shape: below |
| `Place [s: shape] beside [t: shape]` | — | N | Position relative to another shape: beside |
| `Icon [name: text]` | shape | N | An icon of the fixed set, 32 units, stroked |
| `Counter [v: number]` | shape | N | A whole number whose digits roll when it changes |
| `Role [s: shape] as [r: text]` | — | N | Type role of a label or counter: `"title"`, `"label"`, `"value"` or `"caption"` |
| `Face [s: shape] as [f: text]` | — | N | Font face of a label or counter: `"sans"` or `"mono"` |
| `Dash [s: shape] on [a: number] off [b: number]` | — | N | Dashed outline; `on 0` draws solid |
| `Bend [s: shape] by [c: number]` | — | N | Later moves of `s` follow an arc bent by `c` (−1..1) of their length |
| `Trail [s: shape] for [d: time]` | — | N | Moving `s` leaves a fading trail `d` long (0s to 2s) |
| `Focus [s: shape] zoom [z: number] over [d: time]` | — | N | Move the camera to `s` at zoom `z` (0.5..4) |
| `MessageColor [m: msg]` | color | N | One of `theme.Message1`..`4`, chosen by the record type of `m` |
| `Center` | shape | N | An invisible point at the scene centre |
| `Orbit [s: shape] around [c: shape] at [f: number]` | — | N | Place on a circle of radius 300 around `c`, at fraction `f` of a turn (0 is 12 o'clock, clockwise) |
| `Flight [s: shape] from [a: shape] to [b: shape] for [m: msg]` | — | N | Move `s` from `a` to `b`, landing when `m` arrives (halfway when dropped) |
| `delete [s: shape]` | — | N | Unexported: delete the shape now (used by `Remove`) |
| `Fade [s: shape] to [o: number]` | — | S | Opacity over 300ms |
| `Pulse [s: shape]` | — | S | Scale to 1.15 and back over 300ms |
| `Travel [s: shape] from [a: shape] to [b: shape] over [d: time]` | — | S | Move along a straight path |
| `Remove [s: shape]` | — | S | Fade out, then delete |
| `Frame` | shape | S | A rect of the canvas inset by `theme.Margin` |
| `Reveal [s: shape]` | — | S | Draw `s` on: its outline, then its fill; a label writes itself |
| `PopIn [s: shape]` | — | S | Grow from 0 past 1.12 to 1 over 400ms |
| `Enter [l: list of shape] every [d: time]` | — | S | `PopIn` each shape, `d` after the one before |

```synod
# package look
# Opacity over 300ms
define Fade [s: shape] to [o: number]
  Tween "opacity" of s to o over 300ms

# Scale to 1.15 and back over 300ms
define Pulse [s: shape]
  Tween "scale" of s to 1.15 over 150ms
  after 150ms
    Tween "scale" of s to 1 over 150ms
```

`std/look/basic` is the default look described in `spec/09`. It is an
ordinary Synod package using the phrases above. Looks and themes beyond the
defaults are in `github.com/synodlang/looks`, not in std (decision 122).
