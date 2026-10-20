# Standard library catalog

The standard library is the module `github.com/synodlang/std`, imported as
`std/<package>`. It is normative: implementations must provide exactly
these phrases with exactly these patterns. Each table lists the pattern,
what it gives back (— for action phrases), where it may be used, and
whether it is a native (N) or written in Synod (S).

Context abbreviations: **node**, **main**, **check**, **look** (`spec/05`).
"any" means every context.

A change to this catalog is not done until `agent-skills` is regenerated
(`synod doc --catalog`, see `implementation/08-agent-skills.md`).

## std/sim — nodes, runs, faults

| Pattern | Gives | Context | Kind | Meaning |
| --- | --- | --- | --- | --- |
| `Seed [n: number]` | — | main | N | Seed the run's PRNG (default 1) |
| `StopAt [t: time]` | — | main | N | End the run at simulated time `t` |
| `Node running [p: package]` | node | main | N | Create and start one node |
| `Cluster of [count: number] running [p: package]` | list | main | N | Create and start `count` nodes |
| `Crash [n: node]` | — | main | N | Stop a node (`spec/07`) |
| `Restart [n: node]` | — | main | N | Restart a crashed node with fresh state |
| `Remove [n: node]` | — | main | N | Remove a node for good |
| `Here` | node | node | N | The current node |
| `Nobody` | node | any | N | The "no node" value |
| `Nodes` | list | main, check, look | N | All nodes not removed, in creation order |
| `NodesRunning [p: package]` | list | main, check, look | N | Nodes running `p` |
| `IsUp [n: node]` | bool | main, check, look | N | Not crashed and not removed |

Identity fields, declared natively on every node and readable in every
context: `sim.Id` (number, 1, 2, … in creation order) and `sim.Name`
(`"n1"`, `"n2"`, …). In node context `sim.Id` alone is the current node's;
anywhere, `sim.Id of n`.

**Node event:** `Delivered [r: msg]`, a record injected with `deliver`.

**Trace events:** `Started`, `Ended`, `Added [n: node]`,
`Removed [n: node]`, `Crashed [n: node]`, `Restarted [n: node]`,
`Changed [f: field] of [n: node] from [old: value] to [new: value]`,
`Injected [r: msg] into [n: node]`, `Fired on [n: node]`,
`Broken [rule: text]`, `Emitted [e: msg] by [n: node]`,
`Failed [error: text]` (`spec/08`).

## std/net — the simulated network

| Pattern | Gives | Context | Kind | Meaning |
| --- | --- | --- | --- | --- |
| `Send [m: msg] to [n: node]` | — | node | N | Send through the network |
| `Latency between [a: time] and [b: time]` | — | main | N | Uniform delay per message (default 10ms–30ms) |
| `Loss [p: number]` | — | main | N | Probability each message is lost (default 0) |
| `Partition [a: list] from [b: list]` | — | main | N | Cut every link between the two groups |
| `Heal` | — | main | N | Remove all partitions |
| `DropNext [type: text] from [a: node] to [b: node]` | — | main | N | Drop the next message of that record type (unqualified name) on that link |

**Node event:** `Received [m: msg] from [a: node]`.

**Trace events:** `Sent [m: msg] from [a: node] to [b: node]`,
`Arrived [m: msg] from [a: node] to [b: node]`,
`Dropped [m: msg] from [a: node] to [b: node] because [r: text]`,
`Partitioned [a: list] from [b: list]`, `Healed`.

## std/time

| Pattern | Gives | Context | Kind | Meaning |
| --- | --- | --- | --- | --- |
| `Now` | time | node, main, check | N | Simulated time since the run started |
| `Every [d: time] [b: block]` | — | node, main | S | Run `b` now and every `d` |

```synod
# package time
define Every [d: time] [b: block]
  b
  after d
    Every d b
```

## std/random — deterministic randomness

| Pattern | Gives | Context | Kind | Meaning |
| --- | --- | --- | --- | --- |
| `Pick [l: list]` | item | node, main | N | A uniformly chosen item; error on an empty list |
| `Between [a: number] and [b: number]` | number | node, main | N | Uniform in [a, b) |
| `TimeBetween [a: time] and [b: time]` | time | node, main | N | Uniform in [a, b), whole microseconds |
| `Chance [p: number]` | bool | node, main | N | True with probability `p` |

## std/lists

| Pattern | Gives | Context | Kind | Meaning |
| --- | --- | --- | --- | --- |
| `Size [l: list]` | number | any | N | Number of items |
| `Append [x: value] to [l: list]` | list | any | N | A copy with `x` at the end |
| `Without [x: value] from [l: list]` | list | any | N | A copy without any item equal to `x` |
| `Take [n: number] from [l: list]` | list | any | N | The first `n` items (fewer if the list is shorter) |
| `Range [a: number] to [b: number]` | list | any | N | Whole numbers from `a` to `b`, inclusive; empty when `a` is more than `b` |
| `Sort [l: list]` | list | any | N | Ascending; items must all be numbers, all times or all text |

## std/maps

| Pattern | Gives | Context | Kind | Meaning |
| --- | --- | --- | --- | --- |
| `Size [m: map]` | number | any | N | Number of keys |
| `Keys [m: map]` | list | any | N | Keys in ascending order |
| `Without [k: value] from [m: map]` | map | any | N | A copy without key `k` |
| `KeysByValue [m: map]` | list | any | N | Keys ordered by ascending value, ties by key |

## std/math

| Pattern | Gives | Context | Kind | Meaning |
| --- | --- | --- | --- | --- |
| `Floor [x: number]` | number | any | N | Round down |
| `Ceil [x: number]` | number | any | N | Round up |
| `Min of [a: number] and [b: number]` | number | any | N | Smaller |
| `Max of [a: number] and [b: number]` | number | any | N | Larger |
| `Remainder of [a: number] by [b: number]` | number | any | N | `a − b × floor(a / b)` |
| `Hash [t: text]` | number | any | N | FNV-1a 64-bit of the UTF-8 bytes, divided by 2⁶⁴: a fraction in [0, 1) |

## std/text

| Pattern | Gives | Context | Kind | Meaning |
| --- | --- | --- | --- | --- |
| `From [v: value]` | text | any | N | Text form: numbers without trailing zeros, durations canonical, nodes by name, `true`/`false` |
| `Join [l: list]` | text | any | N | Concatenate a list of texts |
| `Size [t: text]` | number | any | N | Number of characters |

## std/disk — storage that survives crashes

| Pattern | Gives | Context | Kind | Meaning |
| --- | --- | --- | --- | --- |
| `Write [v: value] at [k: text]` | — | node | N | Store `v` durably on this node (instantly; records `disk.Written`) |
| `Read [k: text] or [d: value]` | value | node | N | The stored value, or `d` when nothing is stored; same type as `d` |

**Trace event:** `Written [k: text] on [n: node]`.

## std/lesson — Predict mode and captions

| Pattern | Gives | Context | Kind | Meaning |
| --- | --- | --- | --- | --- |
| `Ask [q: text]` | — | node, main | S | Emit `Asked`; the player pauses and asks the viewer to predict |
| `Note [t: text]` | — | node, main | S | Emit `Noted`; the player shows a caption |

```synod
# package lesson
Asked has
  set Question to ""

Noted has
  set Text to ""

define Ask [q: text]
  emit (message Asked with Question q)

define Note [t: text]
  emit (message Noted with Text t)
```

## std/theme — named colors

`Ink`, `Paper`, `Line`, `Quiet`, `Accent`, `Good`, `Warning`, `Critical`:
one-word color phrases, look context. Default values (the Synod brand):

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

## std/look — shapes, motion, layout (look context only)

| Pattern | Gives | Kind | Meaning |
| --- | --- | --- | --- |
| `Circle [r: number]` | shape | N | A circle of radius `r` points |
| `Rect [w: number] by [h: number]` | shape | N | A rounded rectangle |
| `Label [t: text]` | shape | N | Text |
| `Line from [a: shape] to [b: shape]` | shape | N | A line that follows both shapes |
| `Group [l: list]` | shape | N | Shapes moved and faded together |
| `Card for [n: node]` | shape | S | The standard node card: rounded rect, name label |
| `[s: shape] Fill [c: color]` | — | N | Fill color (morphs) |
| `[s: shape] Stroke [c: color]` | — | N | Outline color (morphs) |
| `Tween [p: text] of [s: shape] to [v: value] over [d: time]` | — | N | The only motion primitive (`spec/09`) |
| `Effect [e: text] on [s: shape]` | — | N | `"glow"`, `"blur"` or `"trail"`, for 600ms |
| `Layout [kind: text]` | — | N | `"ring"`, `"row"` or `"grid"` for node cards |
| `Place [s: shape] above [t: shape]` | — | N | Position relative to another shape; likewise `below`, `beside` |
| `Place [s: shape] below [t: shape]` | — | N | |
| `Place [s: shape] beside [t: shape]` | — | N | |
| `Center` | shape | N | An invisible point at the scene centre |
| `Orbit [s: shape] around [c: shape] at [f: number]` | — | N | Place on a circle of radius 300 around `c`, at fraction `f` of a turn (0 is 12 o'clock, clockwise) |
| `Fade [s: shape] to [o: number]` | — | S | Opacity over 300ms |
| `Pulse [s: shape]` | — | S | Scale to 1.15 and back over 300ms |
| `Travel [s: shape] from [a: shape] to [b: shape] over [d: time]` | — | S | Move along a straight path |
| `Remove [s: shape]` | — | S | Fade out, then delete |

```synod
# package look
define Fade [s: shape] to [o: number]
  Tween "opacity" of s to o over 300ms

define Pulse [s: shape]
  Tween "scale" of s to 1.15 over 150ms
  after 150ms
    Tween "scale" of s to 1 over 150ms
```

`std/look/basic` is the default look described in `spec/09`. It is an
ordinary Synod package using the phrases above.
