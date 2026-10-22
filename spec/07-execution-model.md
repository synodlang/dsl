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

The run ends when the queue is empty (`sim.Ended` reason `"empty"`, at the
time of the last step) or when the next event is later than `sim.StopAt`
(reason `"stop"`, at `sim.StopAt`). Events at exactly `sim.StopAt` run. An
event later than 10 minutes of simulated time (and than `sim.StopAt`, if
set) fails the run with `time-limit` at 10 minutes.

- Events whose node is down, removed or in an older incarnation (timers,
  starts, deliveries to a crashed node) are discarded without a step.
  A message arriving at a crashed node is a step: `net.Dropped`, reason
  `"crashed"`.
- More than 100000 steps at one instant is a `run-limit` error, as are
  call depth over 500 and a collection or text over 1000000 items.
- `sim.StopAt` is a time of at least 0; the last call wins, and a time in
  the past ends the run after the current step.
- A run-time error in `define main`, a step or an `always` ends the run.

### Order at time 0

1. `define main` runs to completion and its `after` blocks are
   scheduled. A node created in main (or later) gets the initial
   `Node has` state of every package it runs **immediately**, so main can
   adjust it (`set server.tcp.State to "LISTEN"`), and a start event is
   queued for it.
2. Each node's start event runs the `define [n: Node] start` of every
   package the node runs that has one, in dependency order, with the
   receiver bound to the node (one step per node, in creation order).

## 7.3 Node events

- `deliver r to n` (main only) records `sim.Injected` and schedules
  `sim.Delivered r` on node `n` at the current time. A delivery to a
  crashed node is lost.
- `net.Send m from a to b` records `net.Sent` and schedules
  `net.Received m from a` on `b` after the network delay, unless the message is lost or the link is
  partitioned, in which case `net.Dropped` is recorded instead (at the time
  it would have arrived).
- A node event runs every matching `when` of every package the node runs,
  in package dependency order, then file order, with the `when`'s
  receiver bound to the node.

## 7.4 Timers

`after d` schedules its block at `now + d` in the same context: on the
receiver's node in node context, or in main. In a phrase without a
receiver that several contexts share (`time.Every`), it schedules on the
node of the running event, or in main. A negative delay is a
`bad-argument` error. The block captures the
**values** of locals and bound names, the receiver included, at
scheduling time. Fields read through the receiver (`n.Heartbeat`) are
read live when it runs. Because the block holds copies, setting a local
captured from outside it is a `bad-place` error, and `result` inside it is
a `context` error. There is no timer
cancellation: check a condition when the timer fires instead.

## 7.5 Crash and restart

- `sim.Crash n`: `n` stops. Its pending timers are discarded. Messages
  arriving while it is down are dropped (`net.Dropped`, reason `"crashed"`).
  Messages it already sent are still delivered.
- `sim.Restart n`: every node field returns to its initial value,
  including fields `define main` changed (main's adjustments are not
  reapplied), `std/disk` contents survive, and each package's `start` runs
  again. Timers
  from before the crash never fire (each node has an incarnation number).

## 7.6 The network

Configured from `main` with `std/net` (see `std/README.md`):

- Delay per message: uniform between `net.Latency between a and b`
  (default 10ms and 30ms). Random delays reorder messages naturally.
- Loss probability `net.Loss p` (default 0).
- Partitions: `net.Partition a from b`, `net.Heal`.
- Targeted faults: `net.DropNext "Accept" from a to b`.
- Sending to yourself goes through the network like any other message.

## 7.7 Determinism

- One PRNG, **PCG32**, seeded with the run's seed: `--seed` if given,
  otherwise `sim.Seed n`, otherwise 1. `n` is a whole number from 0 to
  2^53. The trace header records the seed in force when `define main`
  returns. Every random decision draws from the PRNG in step order, and
  each consumes exactly one draw whatever its arguments: every `random.*`
  call (except `random.Pick` of an empty list, an error) and the latency of
  every `net.Send`, drop or not, even with a fixed latency; the loss test
  only when loss is above 0. Partition and targeted drops draw nothing.
- Evaluation order is left to right: a phrase's receiver, then its items
  in the order written, then its blocks; operands left to right (`and` and
  `or` stop early); record settings in the record's field order.
- Numbers are IEEE-754 float64; implementations must not use fused
  multiply-add or other non-reproducible math.
- Map iteration is in ascending key order. Keys order by type (text by
  byte, number, time, node by creation), lists and maps item-wise.
- Time `×` or `÷` a number rounds down to a whole microsecond. `-0` is
  `0`; a result that is not finite, or a time that overflows, is a
  `number-range` error.
- The same program, seed and Synod version must produce a byte-identical trace.

## 7.8 Run-time errors

A run-time error (missing key, type mismatch, division by zero, sending
to `sim.Nobody`…) stops the run and is recorded as the last trace event
`sim.Failed` with the error code, message and source location. The location
is in user code: an error raised inside std is placed at the call that entered
std (`spec/11` §11.3). There is
no `sim.Ended` after it. Everything up to it can still be played. Codes:
`spec/11` §11.3.
