# 9. The visual layer

Shapes, motion and effects are phrases built on a few natives in
`std/look`, exactly like behaviour. Looks live in ordinary packages and
run in **look context** (`spec/05`): they observe the trace, draw, and
can never change what happens.

## 9.1 Observers

A `when` on a trace event is an observer. It runs once per occurrence of
the event, on the presentation clock, and sees exported state **as it was
at that point of the trace**: the state after that fact (`spec/05`). A
timer (`after`) sees the state as of the last fact applied before its
presentation time.

```synod
when (sim.Crashed n)
  look.Fade (card of n) to 0.3
```

## 9.2 Drawing

```ebnf
drawing    = "draw" arg "as" word [ "for" arg ] [ block ] ;
attachment = word "of" arg ;     (* look context: a shape attached with draw … for *)
```

- `draw (look.Circle 8) as dot` creates a shape named `dot`, a local
  visible for the rest of the enclosing block. A draw name need not be
  used (drawing is itself the effect), and drawing the same name again in
  the same scope rebinds it; that is not shadowing. A new shape starts at the scene centre
  (`look.Center`) until it is placed.
- `for v` **attaches** the shape to `v`, a node or a text or number key,
  under that name, for the whole run: any observer can then use
  `card of n` or `tag of "user:1"`.
- **Replacement.** Drawing a name that is already attached to `v` replaces
  the old shape. The new shape takes the old one's `x`, `y`, `scale`,
  `rotation` and `opacity`, and `fill`, `stroke` and `text` where both kinds
  have them; Place constraints and the layout slot move to the new shape.
  If the old shape was drawn in the same instant, it is deleted silently and
  the new one shows at once. Otherwise the old shape fades out over 300ms
  and is then deleted, and the new one fades in over 300ms. A replacement is
  not brand-new (the morph rule, §9.3).
- The optional block is **reactive**: its statements are re-run whenever
  any state they read changes, and the renderer morphs from the old look
  to the new one. This gives the 3Blue1Brown transform feel without
  writing animation code. The read-set is replaced on every run. It holds
  the pairs (node, field), (node, up) and node membership, and
  `sim.StateRows n` reads every exported field of `n`. Instances re-run in
  creation order, after the observers of the fact. Shapes drawn without
  `for` during a reactive run belong to the instance and are deleted
  before it re-runs.
- **Deletion.** A deleted shape stays attached as a **tombstone**: every
  look phrase on it does nothing, so a late `card of n` never fails because
  a node left. Deleting a group deletes its members. The only deletion is
  the unexported `look.delete`; `look.Remove` is the public spelling
  (decision 64).
- **Groups.** `look.Group {a, b}` makes a shape whose members' `x` and `y`
  are relative to the group. The group starts at its first member's
  position. A shape belongs to at most one group (a second is
  `bad-argument`). `Fill` and `Stroke` on a group apply to its non-label
  members; opacity and position compose. Group members take no part in
  layout.
- **Place is a constraint.** `look.Place s above|below|beside t` gives `s`
  an offset from `t` and `s` follows every `x` and `y` change of `t` with the
  same times, until `s` itself is moved (`Tween` of `x` or `y`, `Orbit`,
  `Flight` or a new `Place`), which replaces or breaks the constraint
  (decision 69). A cycle is `bad-argument`. Sizes: circle `2r` by `2r`; rect
  `w` by `h`; label `8` per character by `18`; group the bounding box of its
  members; line and `look.Center` zero; gap 8. `above`: `dy = -(ht/2 + 8 +
  hs/2)`; `below`: `dy = ht/2 + 8 + hs/2`; `beside`: `dx = wt/2 + 8 + ws/2`.
- **Pinning.** `Tween` of `x` or `y`, `Place` (as the moved shape),
  `Orbit`, `Flight` or `Travel` on a node card takes it out of the layout
  for good (§9.4).
- **Defaults.** The scene is 1600 by 1000, origin top-left, `y` down. A new
  shape sits at the centre `(800, 500)` with opacity 1, scale 1, rotation 0.
  A circle and a rect have no fill and no stroke; a rect has corner radius
  `theme.CornerRadius`; a label and a counter are filled `theme.Ink`, with
  role `label` and face `sans` (§9.5); an icon is stroked `theme.Ink` and
  has no fill; a line is stroked `theme.Line`, drawn at opacity
  `theme.LinkOpacity`, and its ends follow the shapes it joins. Every shape
  starts with `draw` 1, `halo` 0, no dash, no bend and no trail (§9.3).

- `card of n` is the **attachment** form (look context only; elsewhere a
  `context` error). The word is a literal, not a name, so a shape local
  named `card` does not interfere, and a phrase head `card` in the current
  package takes the sentence instead (`spec/03` §3.3). It reads
  the shape attached to `n` under `card`; a missing attachment is the
  run-time error `missing-key`. It is not a field: attachments
  belong to the scene, and their keys may also be text or numbers
  (`tag of "user:1"`).

```synod
when (sim.Added [n: raft.Node])
  draw (look.Card for n) as card for n
    look.Fill card with (RoleColor n.raft.Role)
```

The typed binding `[n: raft.Node]` (`spec/05` §5.6) gives `n` the fields
of `raft`.

## 9.3 Motion

- `look.Tween [p: text] of [s: shape] to [v: value] over [d: time]` is the
  only motion primitive. Properties: `"x"`, `"y"`, `"scale"`, `"opacity"`,
  `"rotation"` (degrees), `"fill"`, `"stroke"`, `"text"`, `"draw"`, `"halo"`,
  `"icon"`, `"count"`. Values: for `"x"` and `"y"` a number or a shape (that
  shape's world position when the tween starts; looks never read
  coordinates, decision 65); `"scale"` a number at least 0; `"opacity"` a
  number in 0..1; `"rotation"` a number; `"fill"` and `"stroke"` a color;
  `"text"` text; `"draw"` and `"halo"` numbers in 0..1; `"icon"` the name of
  an icon of the set below, on an icon; `"count"` a number at least 0, on a
  counter. Any other property or value, `"icon"` on a shape that is not an
  icon, `"count"` on a shape that is not a counter, or a negative duration,
  is `bad-argument`.
  - `"draw"` is how much of a shape is drawn: 0 draws nothing and 1 draws all
    of it. An outline is drawn first and then the fill; a label shows its
    first characters; a line grows from its `from` end toward its `to` end.
    `"halo"` is a sustained glow around the shape, 0 for none. `"icon"`
    morphs an icon into another of the set. `"count"` is the number a counter
    shows, rounded to a whole number, its digits rolling as it changes. On a
    group, `"draw"` and `"halo"` apply to every member.
- **Icons.** The icon set is `server`, `database`, `disk`, `user`, `client`,
  `clock`, `lock`, `mail`, `check`, `cross`, `warning`, `crown`, `bolt`,
  `cloud`, `key`, `eye`. `look.Icon [name: text]` makes a stroked icon 32
  units wide and high; a name outside the set is `bad-argument`. The contours
  of each icon are normative in `implementation/icons.json`.
  `look.Counter [v: number]` makes a counter showing the whole number nearest
  `v`; a negative `v` is `bad-argument`.
- **One motion curve per program.** Easing is the speed curve of a movement.
  Tween takes none: every animation uses the curve of the program's theme,
  `theme.Easing` (§9.5). Under `"smooth"` it is smootherstep
  `6t⁵ − 15t⁴ + 10t³`. Under `"spring"` the tracks `"x"`, `"y"`, `"scale"`,
  `"rotation"` and the camera use the back-out curve `E(u) = 1 + (c+1)(u−1)³ +
  c(u−1)²` with `c = 1.70158`, and every other track (`"opacity"`, `"fill"`,
  `"stroke"`, `"draw"`, `"halo"`, `"count"`) uses smootherstep, so no color or
  opacity is ever extrapolated. All packages move alike within a run
  (decision 20).
- **Supersede.** A tween on a property cuts the part of a running tween on
  that property after its start (decision 67). A cut number keeps its
  interpolated value at that moment, rounded to 0.01; a cut color or text
  snaps to its target. A tween to the current value changes nothing.
- **The morph rule.** `Fill`, `Stroke`, `Place`, `Orbit`, layout moves and
  reactive property changes apply at once to a brand-new shape (drawn in this
  instant, not a replacement); on any other shape they tween over 300ms.
  `Tween` keeps its own duration.
- `look.Flight [s: shape] from [a: shape] to [b: shape] for [m: msg]` moves
  `s` from `a` to `b` on a path with the motion curve of the program: a straight line if `s` has
  no bend, else the arc of its bend (below). It starts now
  and lands at the presentation slot of the `net.Arrived` of `m`, or stops
  halfway at the slot of its `net.Dropped`, so no look-ahead is needed
  (decision 63). If `m` has already landed, `s` jumps to the end point; if it
  never lands (the run stopped), `s` stops halfway at the scene's last slot.
  `m` must be bound by a net event (`net.Id m`), else `bad-argument`; a later
  `x` or `y` tween on `s` cancels the pending flight.
- `look.Effect [name: text] on [s: shape]` starts the effect `name` on `s` now
  and clears it after its duration, scaled by pace. The effects and their
  durations are `"glow"` 600ms, `"indicate"` 600ms, `"flash"` 500ms,
  `"circumscribe"` 900ms, `"ripple"` 700ms and `"sweep"` 800ms; any other
  name is `bad-argument`. `"glow"` lights the shape; `"indicate"` swells it
  and tints its fill toward `theme.Warning`; `"flash"` throws rays out from
  it; `"circumscribe"` draws an outline round it; `"ripple"` sends a ring
  out from it; `"sweep"` runs a bright segment along its outline or line.
  The frames of each effect are fixed functions of the scene, given in
  `implementation/04-formats.md`. Readers draw an `effect` value they do not
  know as `none`, so effects can be added within scene format 1.
- **Arcs and trails.** `look.Bend [s: shape] by [c: number]` makes later moves
  of `s` (`Tween` of `"x"` and `"y"`, `Flight`, `Travel`) follow an arc: the
  path bulges to the left of the direction of motion by `c` times its length,
  `c` in −1..1, and 0 is straight. `look.Trail [s: shape] for [d: time]`
  makes `s` leave a fading trail `d` long while it moves, `d` in 0s..2s, and
  0s is no trail. A value outside its range is `bad-argument`. Both stay in
  force until set again.
- **Dash.** `look.Dash [s: shape] on [a: number] off [b: number]` draws the
  outline of `s` as dashes `a` units long with gaps `b` units long; `a` and
  `b` are at least 0, and `on 0` draws the outline solid.
- **Camera.** `look.Focus [s: shape] zoom [z: number] over [d: time]` moves
  the camera to the world position `s` has when the call starts, at zoom `z`
  in 0.5..4, over `d`. The camera starts at the scene centre with zoom 1. A
  zoom outside its range or a negative duration is `bad-argument`.
- `look.Role [s: shape] as [r: text]` sets the type role of a label or counter
  to `"title"`, `"label"`, `"value"` or `"caption"`, and `look.Face [s: shape]
  as [f: text]` sets its face to `"sans"` or `"mono"` (§9.5). Any other
  value, or a shape that is not a label or counter, is `bad-argument`.
- `look.Orbit` puts a shape on the circle of radius 300 around the centre at
  a fraction `f` of a turn: `x = cx + 300·sin(2πf)`, `y = cy − 300·cos(2πf)`,
  and pins it.
- Everything else (`look.Fade`, `look.Pulse`, `look.Travel`,
  `look.Remove`, `look.Reveal`, `look.PopIn`, `look.Enter`) is defined in
  Synod on top of `look.Tween`, `after`, and the unexported `look.delete`.
  `Tween` of `"x"` or `"y"` accepts a shape, which is what lets `Travel` be
  written in Synod.
  - `look.Reveal [s: shape]` draws `s` on: it sets `"draw"` to 0 and tweens it
    to 1, so the outline comes first, then the fill, and a label writes
    itself.
  - `look.PopIn [s: shape]` grows `s` from scale 0 past 1.12 to 1 over 400ms.
  - `look.Enter [l: list of shape] every [d: time]` pops in each shape of `l`,
    each `d` after the one before it, the first at once.
- `look.Frame` is a rect of the whole scene inset by `theme.Margin` on every
  side, with no fill and no stroke.
- `look.MessageColor [m: msg]` is one of `theme.Message1` to `theme.Message4`:
  number `1 + (h mod 4)`, `h` being the FNV-1a 32-bit hash of the record type
  name of `m` as the trace writes it.
- Numbers the builder computes (interpolations, layout, orbit, offsets) are
  rounded to 0.01; numbers a look passes in are kept. Sine and cosine are
  computed by a deterministic routine, never by the platform's.

## 9.4 Layout

Node cards are placed by layout, never by coordinates in behaviour code.
`look.Layout "ring"` (also `"row"`, `"grid"`; any other kind is
`bad-argument`) arranges every shape attached to a node under the name
`card`, for nodes not removed, whose card is not deleted or pinned, in node
id order. With `N` members and index `i` from 0:

- `ring`: fraction `i/N`, radius `300·F`, 0 at 12 o'clock, clockwise (the
  Orbit formula with that radius).
- `row`: `x = 800 + (2i − (N−1))·80·F`, `y = 500`.
- `grid`: `c` is the smallest integer with `c² ≥ N`, `r = ceil(N/c)`,
  `x = 800 + (2·col − (c−1))·100·F`, `y = 500 + (2·row − (r−1))·80·F`.

`F` is the layout scale of the theme, `theme.Fill` (§9.5).

After each activation in which membership or the layout kind changed the
layout re-flows: new cards are placed at once, the others morph (§9.3). Other
shapes are positioned relative to cards (`look.Place … above`, `below`,
`beside`, §9.2). A card that is pinned (§9.2) stays out of the layout.

## 9.5 Themes, colors and type

A **theme** (`spec/06` §6.7) gives the skin of a run: colors, type, stroke,
sketch, glow, shadow, vignette, easing, drift, node shape and fill, corner
radius, link opacity, margin and layout scale. A look gives what is drawn
and how it moves (decision 115). Looks read theme values as `theme.<Name>`
(`std/theme`), so a look is written once and works under every theme.

- Colors are named by zero-hole phrases. A color literal is allowed only
  as the `result` of such a phrase:
  ```synod
  define Accent
    result #F4A340
  ```
- A color's name is the phrase that returns it. A `std/theme` phrase keeps
  its bare name (`Accent`); any other is `<qualifier>.<Name>` with the
  qualifier as in the trace (`raftlook.Hot`). The scene lists colors by name
  and hex (`implementation/04-formats.md`).
- A theme defines exactly the 34 names below, each of the type and in the
  domain shown. The Default column is `std/theme/default`, the Synod brand
  (navy ground, pale nodes, amber accent; see `assets/icon.svg`); the scene
  column says where the value goes (`implementation/04-formats.md`). The
  scene lists a style value only when it differs from the default theme.

| Name | Type, domain | Default | In the scene |
| --- | --- | --- | --- |
| `Paper` | color | `#151A2E` | theme |
| `Ink` | color | `#EDE9E1` | theme |
| `Line` | color | `#3A4366` | theme |
| `Quiet` | color | `#8A90A8` | theme |
| `Accent` | color | `#F4A340` | theme |
| `Good` | color | `#5CC8B8` | theme |
| `Warning` | color | `#E8C35A` | theme |
| `Critical` | color | `#E86F5A` | theme |
| `Message1` | color | `#F4A340` | theme, when used |
| `Message2` | color | `#7FB2F0` | theme, when used |
| `Message3` | color | `#C79BEA` | theme, when used |
| `Message4` | color | `#F08FB0` | theme, when used |
| `Vignette` | number, 0..1 | 0 | `style.vignette` |
| `Sketch` | number, 0..1 | 0 | `style.sketch` |
| `Glow` | number, 0..48 | 24 | `style.glow` |
| `Shadow` | number, 0..1 | 0 | `style.shadow` |
| `StrokeWidth` | number, 0.5..6 | 2 | `style.stroke` |
| `CornerRadius` | number, 0..40 | 12 | baked into rects |
| `NodeShape` | text, `"card"` or `"circle"` | `"card"` | baked into `look.Card` |
| `NodeFill` | text, `"solid"` or `"outline"` | `"solid"` | baked into `look.Card` |
| `LinkOpacity` | number, 0..1 | 1 | baked into `look.Line` |
| `Easing` | text, `"smooth"` or `"spring"` | `"smooth"` | `style.easing` |
| `Drift` | number, 0..24 | 0 | `style.drift` |
| `Sans` | text, a family of the font set | `"Inter"` | `style.sans` |
| `Mono` | text, a family of the font set | `"JetBrains Mono"` | `style.mono` |
| `TitleSize` | number, 12..64 | 28 | `style.titleSize` |
| `LabelSize` | number, 10..40 | 16 | `style.labelSize` |
| `ValueSize` | number, 10..40 | 15 | `style.valueSize` |
| `CaptionSize` | number, 12..48 | 20 | `style.captionSize` |
| `TitleWeight` | number, 400, 500, 600 or 700 | 600 | `style.titleWeight` |
| `LabelWeight` | number, 400, 500, 600 or 700 | 500 | `style.labelWeight` |
| `Tracking` | number, −0.05..0.2 | 0 | `style.tracking` |
| `Margin` | number, 0..100 | 4 | baked into `look.Frame` |
| `Fill` | number, 0.5..1.5 | 1 | baked into the layout (§9.4) |

- **Units.** `Glow` is a blur radius in scene units; `Shadow` and `Vignette`
  are opacities of black; `Sketch` is the strength of the hand-drawn wobble
  of outlines, 0 for none and 1 for full; `Drift` is the amplitude in scene
  units of a slow camera sway, 0 for none; `Tracking` is letter spacing in
  em; `Fill` scales the layout (§9.4). `LinkOpacity` is the opacity of the
  first key of a `look.Line`.
- **Easing** is the program's motion curve (§9.3).
- **Fonts.** The font set is closed: Inter, JetBrains Mono, Source Serif 4
  and Shantell Sans (handwritten), each under the SIL Open Font License 1.1
  and variable in weight, so every weight 400 to 700 exists. `Sans` and
  `Mono` name only these four; any other family is `bad-theme`. A hand-drawn
  text is a theme choice (`Sans "Shantell Sans"`), never a face.
- **Type roles and faces.** A label or counter has a **role** (`title`,
  `label`, `value` or `caption`; default `label`) and a **face** (`sans` or
  `mono`; default `sans`). The role gives the size, `TitleSize`, `LabelSize`,
  `ValueSize` or `CaptionSize`, and the weight: `TitleWeight` for `title`,
  `LabelWeight` for the other three. The face gives the family, `Sans` or
  `Mono`. Text is centered and set with `Tracking`. A look never gives a
  size, weight or family (decision 72); it sets a role or a face with
  `look.Role` and `look.Face` (§9.3).
- `Place` sizes (§9.2) do not depend on role, face or font: the builder
  has no font metrics.
- `std/theme` names every row of the table as a look-context phrase
  (`theme.Ink`, `theme.Sketch`, …), whose value is the program theme's. As
  they are look-context only (`spec/05`), no node, main or check code reads a
  theme, so the trace of a run is the same under every theme.

## 9.6 The presentation clock

The scene builder maps trace order onto screen time. Presentation times are
whole milliseconds. Pace is an integer in thousandths, 250 to 4000
(1000 is 1×); `scale(ms) = ms·1000/pace`, rounded down. Every look duration
(`after`, `Tween`, and the builder's constants below) is scaled by pace
(decision 66); an `after` duration in microseconds becomes milliseconds by
rounding down. Normative rules:

1. **Order is preserved.** If fact A precedes fact B in the trace, A's
   animation starts no later than B's.
2. **Flights are readable.** A message flight (from `net.Sent` to
   `net.Arrived` or `net.Dropped`) lasts at least `scale(400ms)` on screen;
   later facts shift to make room.
3. **Idle gaps are compressed.** The simulated gap between two facts shows as
   `gap × 8` milliseconds (the gap in simulated microseconds times 8 over 1000, rounded down),
   but at most `scale(600ms)`.
4. **Tweens finish.** A fact waits until the shapes it touches are free of
   tweens and morphs already running on them (effects, the camera and pending
   flights do not count; the tweens of `"draw"`, `"halo"`, `"icon"` and
   `"count"` do, while `Bend`, `Trail`, `Dash`, `Role` and `Face` set
   properties that do not tween, so they never count). The shapes it touches are the live shapes attached under any
   name to any node of the fact (`node`, `from`, `to`, and both lists of
   `net.Partitioned`), plus the owner shapes of reactive blocks whose
   read-set meets the fact's changes. A group counts its members.
5. **Pace.** A global pace factor (0.25× to 4×) scales everything.
6. **Proportional mode** (a player toggle, `--mode proportional`) instead
   maps screen time linearly to simulated time, for lessons where timing is
   the point: tail latency, racing timeouts, clock skew. The slot of a fact
   is `t · 30000 / End`, `End` being the time of the last fact (0 if `End` is
   0), then scaled by pace and never earlier than the previous slot; equal
   slots are allowed. Only rules 1 and 5 apply.

In ordered mode the first fact starts at 0; each later fact starts at least
`scale(30ms)` after the previous one. The slot is the largest of: the
previous slot plus the compressed gap, the end of the busy shapes of rule 4,
the send slot plus `scale(400ms)` for an arrival or drop, and the previous
slot plus `scale(30ms)`.

An `after` inside an observer counts presentation time from the moment its
observer starts. Timers interleave with facts by presentation time: a timer
due no later than the next fact's slot runs first, seeing the state as of the
last fact applied before it. After the last fact, timers due within
`scale(2s)` of the last slot run; later ones are dropped. More than 100000
timer firings at one instant is `run-limit`. The scene's duration is the
latest of the last slot, the last key and the last marker.

## 9.7 The default look

`std/look/basic` is used when a program uses no look. A program **uses a
look** when `std/look` is in its package graph (`spec/06` §6.6). Otherwise
the observers of `std/look/basic` run before the program's own. Nodes are
cards on a ring labelled with the node's name (`sim.Name n`), each card lists the node's exported
state fields as `name value` rows that morph on `sim.Changed`, messages
are dots that fly between cards and are labelled with their record type,
dropped messages fade midway, crashed nodes grey out, `sim.Broken`
flashes the whole scene border in `theme.Critical`, and `lesson.Noted`
shows a caption. `std/look/basic` is themable: it draws every color, size, shape and font
through the theme (§9.5). Under `std/theme/default` its scene is the one
described above, unchanged; under another theme it is the same look
reskinned. Look packages may `use std/look/basic` and add to it.
