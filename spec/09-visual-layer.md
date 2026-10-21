# 9. The visual layer

Shapes, motion and effects are phrases built on a few natives in
`std/look`, exactly like behaviour. Looks live in ordinary packages and
run in **look context** (`spec/05`): they observe the trace, draw, and
can never change what happens.

## 9.1 Observers

A `when` on a trace event is an observer. It runs once per occurrence of
the event, on the presentation clock, and sees exported state **as it was
at that point of the trace**.

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
  `card of n` or `tag of "user:1"`. Drawing a name that is already
  attached to `v` replaces the old shape with a morph.
- The optional block is **reactive**: its statements are re-run whenever
  any state they read changes, and the renderer morphs from the old look
  to the new one. This gives the 3Blue1Brown transform feel without
  writing animation code.

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
  `"rotation"` (degrees), `"fill"`, `"stroke"`, `"text"`.
- **One motion curve.** Easing is the speed curve of a movement. Tween takes
  none: every animation uses the house curve, smootherstep
  `6t⁵ − 15t⁴ + 10t³`, so all packages move alike.
- Everything else (`look.Fade`, `look.Pulse`, `look.Travel`,
  `look.Remove`) is defined in Synod on top of `look.Tween` and `after`.

## 9.4 Layout

Node cards are placed by layout, never by coordinates in behaviour code.
`look.Layout "ring"` (also `"row"`, `"grid"`) arranges every shape
attached to a node under the name `card`; when nodes are added or removed,
the layout re-flows with a morph. Other shapes are positioned relative to
cards (`look.Place … above`, `below`, `beside`).

## 9.5 Colors and type

- Colors are named by zero-hole phrases. A color literal is allowed only
  as the `result` of such a phrase:
  ```synod
  define Accent
    result #F4A340
  ```
- `std/theme` names the standard palette: `theme.Ink`, `theme.Paper`,
  `theme.Line`, `theme.Quiet`, `theme.Accent`, `theme.Good`,
  `theme.Warning`, `theme.Critical`. The default theme is the Synod brand
  (navy ground, pale nodes, amber accent; see `assets/icon.svg`).
- Text uses the player's two fonts: a sans for labels and a mono for
  values. Looks choose size and weight only.

## 9.6 The presentation clock

The scene builder maps trace order onto screen time. Normative rules:

1. **Order is preserved.** If fact A precedes fact B in the trace, A's
   animation starts no later than B's.
2. **Flights are readable.** A message flight (from `net.Sent` to
   `net.Arrived` or `net.Dropped`) lasts at least 400ms on screen; later
   facts shift to make room.
3. **Idle gaps are compressed.** A stretch of simulated time with no facts
   takes at most 600ms on screen.
4. **Tweens finish.** A fact on a shape waits for tweens already running
   on that shape.
5. **Pace.** A global pace factor (0.25× to 4×) scales everything.
6. **Proportional mode** (a player toggle) instead maps screen time
   linearly to simulated time, for lessons where timing is the point:
   tail latency, racing timeouts, clock skew.

An `after` inside an observer counts presentation time from the moment
its observer starts.

## 9.7 The default look

`std/look/basic` is used when a program uses no look: nodes are cards on
a ring labelled with the node's name (`sim.Name n`), each card lists the node's exported
state fields as `name value` rows that morph on `sim.Changed`, messages
are dots that fly between cards and are labelled with their record type,
dropped messages fade midway, crashed nodes grey out, `sim.Broken`
flashes the whole scene border in `theme.Critical`, and `lesson.Noted`
shows a caption. Look packages may `use std/look/basic` and add to it.
