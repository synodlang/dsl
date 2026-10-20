# 7. Execution model

## 7.1 Two clocks

| Clock | Drives | Used by |
| --- | --- | --- |
| **Simulated** | behaviour: delays, timeouts, `after`, `time.Every`, `time.Now` | node and main context |
| **Presentation** | the screen: tweens, `after` inside observers | look context |

A handler's `after` uses the clock of the context it runs in. The
simulated clock has microsecond resolution.

## 7.2 The event loop

The simulator keeps one queue of pending events ordered by
**(simulated time, sequence number)**. The sequence number increases with
every insertion, so events at the same time run in the order they were
scheduled. One **step** is:

1. Pop the first event.
2. Run it to completion (handlers never interleave).
3. Append the trace events it produced.
4. Evaluate every active `always`.

The run ends when the queue is empty, when `sim.StopAt` is reached, or
after 10 minutes of simulated time (an error is reported).

### Order at time 0

1. `main` runs to completion. Its `after` blocks are scheduled. A node
   created in `main` (or later) gets its initial `node has` state
   **immediately**, so `main` can adjust it (`set tcp.State of server to
   "LISTEN"`), and a startup event is queued for it.
2. Each node's startup event runs its packages' startup statements, in
   dependency order (one step per node, in creation order).

## 7.3 Node events

- `deliver r to n` (main only) records `sim.Injected` and schedules
  `sim.Delivered r` on node `n` at the current time. A delivery to a
  crashed node is lost.
- `net.Send m to n` records `net.Sent` and schedules `net.Received m` on
  `n` after the network delay, unless the message is lost or the link is
  partitioned, in which case `net.Dropped` is recorded instead (at the time
  it would have arrived).
- A node event runs every matching `when` of every package the node runs,
  in package dependency order, then file order.

## 7.4 Timers

`after d` schedules its block at `now + d` on the same node (or in main).
The block captures the **values** of locals and bound names at scheduling
time; state fields are read live when it runs. There is no timer
cancellation: check a condition when the timer fires instead.

## 7.5 Crash and restart

- `sim.Crash n`: `n` stops. Its pending timers are discarded. Messages
  arriving while it is down are dropped (`net.Dropped`, reason `"crashed"`).
  Messages it already sent are still delivered.
- `sim.Restart n`: every node state field returns to its initial value,
  `std/disk` contents survive, and startup statements run again. Timers
  from before the crash never fire (each node has an incarnation number).

## 7.6 The network

Configured from `main` with `std/net` (see `std/README.md`):

- Delay per message: uniform between `net.Latency between a and b`
  (default 10ms and 30ms). Random delays reorder messages naturally.
- Loss probability `net.Loss p` (default 0).
- Partitions: `net.Partition a from b`, `net.Heal`.
- Targeted faults: `net.DropNext Accept from a to b`.
- Sending to yourself goes through the network like any other message.

## 7.7 Determinism

- One PRNG, **PCG32**, seeded by `sim.Seed n` (default seed 1). Every
  random decision (`random.*`, latency, loss) draws from it in step order.
- Numbers are IEEE-754 float64; implementations must not use fused
  multiply-add or other non-reproducible math.
- Map iteration is in ascending key order.
- The same program, seed and Synod version must produce a byte-identical trace.

## 7.8 Run-time errors

A run-time error (missing key, type mismatch, division by zero, sending
to `sim.Nobody`…) stops the run and is recorded as the last trace event
`sim.Failed` with the error and source location. Everything up to it can
still be played.
