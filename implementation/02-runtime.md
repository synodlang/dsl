# Runtime / simulator (`compiler/internal/sim`)

A tree-walking interpreter over the IR. Speed matters less than
determinism and clarity.

## Core structures

```go
type Sim struct {
    Now      Micros            // simulated clock, int64 microseconds
    Queue    EventHeap         // ordered by (At, Seq)
    Seq      uint64
    RNG      *PCG32            // --seed, else sim.Seed, else 1
    Nodes    []*Node           // creation order; index+1 = sim.Id
    Net      NetConfig         // latency, loss, partitions, targeted drops
    Trace    *TraceWriter
    Checks   []Invariant       // active `always`, with last value
}

type Node struct {
    ID, Name     string / int
    Running      []*Package    // dependency order
    State        map[*Package]*Record   // per-package node state
    Disk         map[string]Value       // survives restart
    Up           bool
    Incarnation  int           // bumped on restart; stale timers are skipped
}

type Event struct {
    At          Micros
    Seq         uint64
    Node        *Node          // nil for main timers
    Incarnation int
    Kind        Start | Deliver | Receive | Timer
    Payload     ...
}
```

## Values

Tagged union: Text, Number (float64), Bool, Time (int64 µs), List,
Map (sorted keys), Node (pointer), Record (type + field array), Shape
(look only). Lists and maps are persistent or copy-on-write, so
`set item` copies cheaply and `after` captures stay correct.

## Run

1. Create the trace; record `sim.Started`.
2. Execute `define main` (main context): sim setup phrases mutate `Sim`;
   `sim.Node`/`sim.Cluster` create nodes, record `sim.Added`, and queue
   a Start event per node at the current time.
3. Loop: pop; skip if the node is down or the incarnation is stale (for
   Receive on a down node, record `net.Dropped … because "crashed"`); set
   `Now`; run; then evaluate invariants; record `sim.Broken` on
   true→false.
4. Stop on empty queue or when the next event is later than `sim.StopAt`
   (events at exactly `sim.StopAt` run); record `sim.Ended`. An event
   later than 10 minutes is a `time-limit` error instead. A run-time error
   records `sim.Failed` as the last event, with no `sim.Ended`.

## Executing a step

- **Creation** (`sim.Node`, `sim.Cluster`) and **restart** initialize every
  run package's `Node has` fields in declaration order, then queue a
  Start event.
- **Start:** for each package the node runs, in dependency order, run its
  `start` (if any) with the receiver bound to the node.
- **Deliver/Receive:** find matching handlers in the node's run packages
  (dependency order, file order, line order); bind the receiver and
  names; run each.
- **Timer:** run the captured block with its captured bindings, which
  include the receiver. The interpreter tracks the node of the running
  event, so `after` in shared code (`time.Every`) schedules on that node.
  Passing a block argument that only runs another block passes that block
  itself, so the nesting of `time.Every` does not grow with its ticks.
- **Restart** also resets fields `define main` set; the demo's setup is
  not reapplied.
- **set** on an exported node field records `sim.Changed` with old and new
  values, but only when the value actually changes.
- **net.Send** (`net.Send m from a to b`; `a` is always the receiver in
  scope): record `net.Sent` with a fresh message id; decide drop
  (partition → targeted → loss, in that order); draw the delay from the
  RNG; enqueue Receive at `Now + delay` (or record `net.Dropped` at that
  time via a dropped-arrival event, so dots fade where they would have
  arrived).
- **Receive** records `net.Arrived` before running handlers.
- **emit** (`emit e by n`) records `sim.Emitted` with the record and the node `n`; in main context `n` is `sim.Nobody`.

## RNG

PCG32 (XSH RR) as in the reference implementation, seeded with
`seed` and stream 54. `random.Between a and b` = `a + (b−a) × u` with
`u = next() / 2³²`. `TimeBetween` rounds down to whole microseconds.
Latency uses `TimeBetween`. Every draw happens in step order and each
random decision consumes exactly one draw (`spec/07` §7.7); no other
source of randomness exists. `math.Hash` is not random: FNV-1a 64 over
the UTF-8 text, then `(h >> 11) / 2^53`.

## Context enforcement at run time

The checker proves the context rules, including `foreign-state`, at compile time. The runtime checks only sending to
`sim.Nobody` (`nobody`); it may assert that the named node of each effect
(`from`, `on`, `by`) equals the node of the running event as an internal
invariant, and a failure there is a compiler bug, not `sim.Failed`.
Errors record `sim.Failed` and stop the run.

## State snapshots for looks

For the scene builder, the runtime also records, per trace event, the
exported state that changed (already in `sim.Changed`). The scene builder
rebuilds full exported state by replaying `sim.Added` initial values and
`sim.Changed` deltas, so the trace alone is enough to drive looks.
`sim.Added` and `sim.Restarted` therefore carry the node's full initial
exported state.
