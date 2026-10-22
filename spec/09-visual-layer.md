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
  12; a label is filled `theme.Ink`; a line is stroked `theme.Line` and its
  ends follow the shapes it joins.

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
  `"rotation"` (degrees), `"fill"`, `"stroke"`, `"text"`. Values: for `"x"`
  and `"y"` a number or a shape (that shape's world position when the tween
  starts; looks never read coordinates, decision 65); `"scale"` a number
  at least 0; `"opacity"` a number in 0..1; `"rotation"` a number;
  `"fill"` and `"stroke"` a color; `"text"` text. Any other property or
  value, or a negative duration, is `bad-argument`.
- **One motion curve.** Easing is the speed curve of a movement. Tween takes
  none: every animation uses the house curve, smootherstep
  `6t⁵ − 15t⁴ + 10t³`, so all packages move alike.
- **Supersede.** A tween on a property cuts the part of a running tween on
  that property after its start (decision 67). A cut number keeps its
  interpolated value at that moment, rounded to 0.01; a cut color or text
  snaps to its target. A tween to the current value changes nothing.
- **The morph rule.** `Fill`, `Stroke`, `Place`, `Orbit`, layout moves and
  reactive property changes apply at once to a brand-new shape (drawn in this
  instant, not a replacement); on any other shape they tween over 300ms.
  `Tween` keeps its own duration.
- `look.Flight [s: shape] from [a: shape] to [b: shape] for [m: msg]` moves
  `s` from `a` to `b` on a straight path with the house curve. It starts now
  and lands at the presentation slot of the `net.Arrived` of `m`, or stops
  halfway at the slot of its `net.Dropped`, so no look-ahead is needed
  (decision 63). If `m` has already landed, `s` jumps to the end point; if it
  never lands (the run stopped), `s` stops halfway at the scene's last slot.
  `m` must be bound by a net event (`net.Id m`), else `bad-argument`; a later
  `x` or `y` tween on `s` cancels the pending flight.
- `look.Effect "glow" on s` lights `s` now and clears it 600ms later; any
  other effect name is `bad-argument`. Glow is the only effect in 0.2;
  readers draw an `effect` value they do not know as `none`, so effects can
  be added within scene format 1 (`implementation/04-formats.md`).
- `look.Orbit` puts a shape on the circle of radius 300 around the centre at
  a fraction `f` of a turn: `x = cx + 300·sin(2πf)`, `y = cy − 300·cos(2πf)`,
  and pins it.
- Everything else (`look.Fade`, `look.Pulse`, `look.Travel`,
  `look.Remove`) is defined in Synod on top of `look.Tween`, `after`, and the
  unexported `look.delete`. `Tween` of `"x"` or `"y"` accepts a shape, which
  is what lets `Travel` be written in Synod.
- Numbers the builder computes (interpolations, layout, orbit, offsets) are
  rounded to 0.01; numbers a look passes in are kept. Sine and cosine are
  computed by a deterministic routine, never by the platform's.

## 9.4 Layout

Node cards are placed by layout, never by coordinates in behaviour code.
`look.Layout "ring"` (also `"row"`, `"grid"`; any other kind is
`bad-argument`) arranges every shape attached to a node under the name
`card`, for nodes not removed, whose card is not deleted or pinned, in node
id order. With `N` members and index `i` from 0:

- `ring`: fraction `i/N`, radius 300, 0 at 12 o'clock, clockwise (the Orbit
  formula).
- `row`: `x = 800 + (2i − (N−1))·80`, `y = 500`.
- `grid`: `c` is the smallest integer with `c² ≥ N`, `r = ceil(N/c)`,
  `x = 800 + (2·col − (c−1))·100`, `y = 500 + (2·row − (r−1))·80`.

After each activation in which membership or the layout kind changed the
layout re-flows: new cards are placed at once, the others morph (§9.3). Other
shapes are positioned relative to cards (`look.Place … above`, `below`,
`beside`, §9.2). A card that is pinned (§9.2) stays out of the layout.

## 9.5 Colors and type

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
- `std/theme` names the standard palette: `theme.Ink`, `theme.Paper`,
  `theme.Line`, `theme.Quiet`, `theme.Accent`, `theme.Good`,
  `theme.Warning`, `theme.Critical`. The default theme is the Synod brand
  (navy ground, pale nodes, amber accent; see `assets/icon.svg`).
- Type. In Synod 0.2 a label is drawn in the player's sans at 16 px, weight
  500, centered; looks cannot choose size, weight or the mono face. Choosing
  them is planned as additive label props (`font`, `size`, `weight`) in
  format 1 (decision 72).

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
   tweens and morphs already running on them (effects and pending flights do
   not count). The shapes it touches are the live shapes attached under any
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
shows a caption. Look packages may `use std/look/basic` and add to it.
