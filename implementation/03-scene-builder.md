# Scene builder (`compiler/internal/scene`)

Turns a trace into a scene: shapes with keyframes on the presentation
clock. It runs the observers of every active look package (`spec/06`
§6.6) in look context.

## Inputs and outputs

- In: the IR (for observers and look phrases), `trace.jsonl`, options
  `pace` (default 1), `mode` (`"ordered"` default, or `"proportional"`).
- Out: `scene.json` (`04-formats.md`), consumed by the `player` library.

## Algorithm

```
state  := empty exported-state store
shapes := empty
cursor := 0ms                      // presentation time
for each trace event e in order:
    apply e to state               // Added/Restarted snapshots, Changed deltas
    slot := schedule(e, cursor)    // presentation start time for e
    for each observer matching e:  // package dependency order, file order;
                                   // a typed binding [n: p.Node] matches only
                                   // nodes whose `runs` list contains p
        run observer at time slot  // draw / tween / after on presentation clock
    re-run reactive draw blocks whose read-set changed in e
    cursor := slot
```

### Scheduling (`ordered` mode)

```
schedule(e, cursor):
    gap  := e.simTime − previous.simTime
    slot := cursor + min(gap × 8 / pace, 600ms / pace)     // idle compression
    slot := max(slot, busyUntil(shapes e touches))          // tweens finish
    if e is net.Arrived or net.Dropped:
        slot := max(slot, sentSlot(e.msgId) + 400ms / pace) // readable flights
    slot := max(slot, cursor + 30ms / pace)                 // strictly increasing
    return slot
```

The constant 8 means 1ms of simulated time shows as 8ms when events are
dense. All constants are named in one place and covered by golden tests.

### `proportional` mode

`slot = simTime × factor / pace`, with `factor` chosen so the whole run
lasts 30s at pace 1. Rules 2 and 4 of `spec/09` §9.6 do not apply.

### Reactive draws

While running a `draw` block, record every (node, field) it reads: its
read-set. After applying each event, re-run blocks whose read-set
intersects the event's changes, at that event's slot, and emit morph
keyframes from the old property values to the new ones over 300ms.

### Messages in the default look

`std/look/basic` draws a dot on `net.Sent` and, on `net.Arrived`, travels it
from sender to receiver between the two slots. The builder therefore keeps
`sentSlot` per message id.

### Lesson pauses

`sim.Emitted lesson.Asked` becomes a `pause` marker in the scene with the
question; the player (in the app or on the site) stops there in Predict mode.

## Output size

Keyframes are only written when a property changes. Each shape has a
stable id, `(observer source location, run index)` hashed, so diffs
between runs stay small and Compare can match shapes across two runs.
