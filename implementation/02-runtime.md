# Runtime / simulator (`compiler/internal/sim`)

A tree-walking interpreter over the IR. Speed matters less than
determinism and clarity.

## Core structures

```go
type Sim struct {
    Now      Micros            // simulated clock, int64 microseconds
    Queue    EventHeap         // ordered by (At, Seq)
    Seq      uint64
    RNG      *PCG32            // seeded by sim.Seed, default 1
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
    Kind        Startup | Deliver | Receive | Timer
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
2. Execute `main` (main context): sim setup phrases mutate `Sim`;
   `sim.Node`/`sim.Cluster` create nodes, record `sim.Added`, and enqueue
   a Startup event per node at the current time.
3. Loop: pop; skip if the node is down or the incarnation is stale (for
   Receive on a down node, record `net.Dropped … because "crashed"`); set
   `Now`; run; then evaluate invariants; record `sim.Broken` on
   true→false.
4. Stop on empty queue, `sim.StopAt`, or 10 minutes; record `sim.Ended`.

## Executing a step

- **Creation** (`sim.Node`, `sim.Cluster`) and **restart** initialize every
  run package's `node has` fields in declaration order, then queue a
  Startup event.
- **Startup:** for each package the node runs, in dependency order, run
  its startup statements.
- **Deliver/Receive:** find matching handlers in the node's run packages
  (dependency order, file order, line order); bind names; run each.
- **Timer:** run the captured block with its captured bindings.
- **set** on an exported node field records `sim.Changed` with old and new
  values, but only when the value actually changes.
- **net.Send:** record `net.Sent` with a fresh message id; decide drop
  (partition → targeted → loss, in that order); draw the delay from the
  RNG; enqueue Receive at `Now + delay` (or record `net.Dropped` at that
  time via a dropped-arrival event, so dots fade where they would have
  arrived).
- **Receive** records `net.Arrived` before running handlers.
- **emit** records `sim.Emitted` with the record and the node.

## RNG

PCG32 (XSH RR) as in the reference implementation, seeded with
`seed` and stream 54. `random.Between a and b` = `a + (b−a) × u` with
`u = next() / 2³²`. `TimeBetween` rounds down to whole microseconds.
Latency uses `TimeBetween`. Every draw happens in step order; no other
source of randomness exists.

## Context enforcement at run time

The checker proves most context rules. The runtime still checks:
writes and reads of node state only on the current node in node context
(`foreign-state`), field reads on nodes that do not run the package
(`not-running`), and sending to `sim.Nobody` (`nobody`). Errors record
`sim.Failed` and stop the run.

## State snapshots for looks

For the scene builder, the runtime also records, per trace event, the
exported state that changed (already in `sim.Changed`). The scene builder
rebuilds full exported state by replaying `sim.Added` initial values and
`sim.Changed` deltas, so the trace alone is enough to drive looks.
`sim.Added` and `sim.Restarted` therefore carry the node's full initial
exported state.
