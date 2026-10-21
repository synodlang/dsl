# Scene builder (`compiler/internal/scene`)

Turns a trace into a scene: shapes with keyframes on the presentation
clock. It runs the observers of every active look package (`spec/06`
§6.6) in look context.

## Inputs and outputs

- In: the IR (observers and look phrases), the facts of the run, and
  options `pace` (thousandths, 250 to 4000, default 1000), `mode`
  (`ordered` default, or `proportional`) and limits (`MaxShapes`,
  `MaxTimersPerInstant`, `MaxDepth`, `MaxKeys`: 1,000,000, 100,000, 10,000,
  10,000,000).
- Out: a `*trace.Scene`, written as `scene.json` (`04-formats.md`),
  consumed by the `player` library; and, on a look fault, a `Failure`
  (`Seq`, `T`, fault) beside the partial scene.
- `Build(Input, Options)`; `Input` holds the program, the `std/look/basic`
  program (only when the program uses no look, `UsesLook`: `std/look` in the
  package graph), the facts and the trace options. `Build(facts)` equals
  `Build(Decode(Encode(facts)))`.

## The look interpreter

A separate interpreter in `internal/scene`, not the simulator's. Pure value
operations (arithmetic with number and time cases and their overflow and
division faults, indexing, map size, plurals, `item of` lookup, list and map
item update) live in `internal/value/ops.go` and are called by both. Non-look
natives are reused through `natives.Table` with a look host (nodes, running
nodes and up-state read from the store, reads recorded); the look-only
natives (`look.*`, `sim.StateRows`, `net.Id`, `net.Type`) are in
`scene.Table`. It supports `set` on locals and items, calls, blocks, `if`,
`each`, `after` (presentation), `draw` and `result`; anything else is a
compiler bug. New value kinds: shape (an index) and color (its name).
Records bound from a net fact carry the message id in `Value.Int`, ignored by
equality. Unexported record fields are masked as `"…"`, as in the trace.

If `std/look/basic` is loaded (`load.Config.ExtraStd`, set by
`driver.Options.DefaultLook`), its observers run first, then the program's, in
dependency then file order. Its IR shares package and record pointers with
the program's, so `lesson.Noted` matches by pointer.

## The store

A replay store holds, per node, its runs, whether it is up, whether it was
removed, and its exported values, filled from the `state` of `sim.Added` and
`sim.Restarted` and updated by `sim.Changed`, `sim.Crashed`, `sim.Removed`.
Observers of a fact see the store after applying it; timers see it as of the
last fact applied before them. Before applying a fact the store computes its
**changes**: `(node, field)` for `sim.Changed`; for `sim.Restarted` every
exported field whose value differs plus `(node, up)`; `(node, up)` for
`sim.Crashed` and `sim.Removed`; and a membership change for `sim.Added` and
`sim.Removed`.

## Scheduling

All times are integer milliseconds; `scale(ms) = ms·1000/pace`, rounded down.
Constants, named in one place and covered by golden tests: `GapScale 8`,
`MaxIdle 600`, `MinFlight 400`, `MinStep 30`, `Morph 300`, `EffectLen 600`,
`TimerTail 2000`, `ProportionalLen 30000`, canvas 1600 by 1000, centre
(800, 500).

```
schedule(e):                                    // ordered mode
    if e is the first fact: return 0
    gapMs := (e.t − prevT) × GapScale / 1000
    s := cursor + scale(min(gapMs, MaxIdle))
    s  = max(s, busyUntil(touched(e)))
    if e is net.Arrived or net.Dropped and sentSlot[id] known:
        s = max(s, sentSlot[id] + scale(MinFlight))
    s  = max(s, cursor + scale(MinStep))
    return s
```

`touched(e)` is the live shapes attached under any name to any node of the
fact, plus the owner shapes of reactive instances whose read-set meets the
fact's changes. `busyUntil(shape)` is the latest end of a tween or morph on
it or its group members; effects and pending flights do not count.
`proportional` mode: `slot = End == 0 ? 0 : e.t × ProportionalLen / End`, then
scaled, then at least the cursor; only rules 1 and 5 of `spec/09` §9.6.

Main loop, for each fact `f` at index `i`:

```
ch := store.changes(f)
loop: s := schedule(f)
      if the next timer is due at or before s: run it at its time; continue
      break
cursor, prevT, slots[i] = s, f.t, s
store.apply(f)
if net.Sent: sentSlot[id] = s
if net.Arrived or net.Dropped: world.land(id, s, dropped)
markers(f, s)
run the observers matching f at now = s
re-run the reactive instances hit by ch, in creation order, at now = s
world.reflow(now = s)
```

After the facts: run timers due within `cursor + scale(TimerTail)` in (time,
creation) order and drop the rest; stop unlanded flights halfway at the
scene's last slot; `duration = max(cursor, last key, last marker)`. A look
`after d` schedules at `now + scale(d/1000)` (d in microseconds). A look
fault stops everything at that fact: the partial scene is finalized as above,
with a `failed` marker.

## Reactive blocks

The block of `draw … as … [for …]` becomes an instance (block, copy of the
frame, owner shape, site) run once with read tracking: `(node, field)`,
`(node, up)`, membership; `sim.StateRows` reads every exported field of its
node. The read-set is replaced on every run, kept in an index from read key
to instances. Unattached shapes drawn during a run belong to the instance and
are deleted, with nested instances, before it re-runs.

## The world

Shapes (circle, rect, label, line, group) with a timeline per property
(`x`, `y`, `scale`, `opacity`, `rotation`, `fill`, `stroke`, `text`,
`effect`). `tween(p, v, t0, d)` removes keys of `p` after `t0`, writes the
interpolated value at `t0` if a running tween was cut (colors and text snap to
the target) or a hold key if `p` has none there, writes `v` at `t0 + d`
(`d = 0` overwrites `t0`), writes nothing for a tween to the current value,
merges keys at the same time and keeps them sorted. Computed numbers are
rounded to 0.01. The morph rule, replacement, tombstones, groups, Place
constraints and sizes are as in `spec/09` §9.2 and §9.3. An effect writes
`effect:"glow"` at now and `effect:"none"` at `now + scale(600)`. A shape
created and deleted in the same instant and never referenced is not written.

**Layout.** Members are shapes attached under `card` to nodes not removed,
not deleted and not pinned, in node id order; positions are as in `spec/09`
§9.4, re-flowed after each activation whose membership or kind changed.
Pinning is permanent.

**Flights.** `look.Flight` writes the start `x` and `y` keys at now (the world
position of the source) and registers a pending flight under the message id.
`land(id, slot, dropped)` writes the end keys at the slot, at the target or
at the midpoint when dropped; a later `x` or `y` tween on the shape cancels
the pending flight; unlanded flights are finalized at the end.

`look.Center` is written, as an empty group, only if a line or group refers
to it.

## Ids

A shape's key is `<path>:<line>:<col>#<run>.<ordinal>`: the activation site
(observer, timer `after`, or reactive re-run `draw`), as a trace path; the run
index counts earlier activations of that site; the ordinal counts shapes
created earlier in the same activation, called phrases included. The id is
`s` plus 12 lowercase hex digits of FNV-1a-64 of the key masked to 48 bits,
assigned at encode time in creation order; on collision the key gets `#2`,
`#3`, …, appended.

## Colors

A color value is named after the zero-hole phrase being executed when its
literal is read: `Accent` for `std/theme` phrases, `<qualifier>.<Name>` for
others (`raftlook.Hot`), with the hex from the literal. The scene's `theme`
lists the eight theme colors in catalog order, then other used names sorted by
byte order. If `std/theme` is not in the IR a constant table is used, kept in
step with `std/theme/theme.synod` by a test.

## Markers

Sorted by (time, fact order): `pause` from `sim.Emitted` with
`std/lesson` `Asked` (matched by path and name), text `Question`; `caption`
from `lesson.Noted`, `Text`; `broken` from `sim.Broken`, the rule text from
`always …` on (without the `path:line:` prefix); `failed` from `sim.Failed`
(`<trace pos>: error[<code>]: <message>`) and from a look fault (`look <trace
pos>: error[<code>]: <message>`). The `trace` map has one `{seq, at}` per fact
processed.

## Determinism and performance

Same input, same bytes, on every platform: no map iteration in output paths,
integer time, an explicit `float64()` around every product before adding (no
fused multiply-add), no wall clock, goroutines or `os` in the package, and
`sin` and `cos` from the package's own routine (an exact reduction of the
fraction of a turn, octants, a polynomial on [0, π/4]; within 1e-12 of the
platform's, pinned by a golden table), never `math.Sin` or `math.Cos`. It
builds for `js/wasm`. Targets: ping with 5000 messages (about 15,000 shapes)
builds and encodes in at most 100ms; `synod run` in at most 200ms. Keys are
fixed structs with a bitmask of set properties; output is encoded into one
byte slice.

## Lesson pauses

`sim.Emitted lesson.Asked` becomes a `pause` marker with the question; the
player (in the app or on the site) stops there in Predict mode.
