# 0. Introduction

## 0.1 What Synod describes

Synod describes **nodes** that keep state, exchange **messages** over a
simulated network, react to **events**, and are checked against
**invariants**. A separate, purely visual layer turns what happened into
an animation.

Three kinds of source live in Synod packages:

- **Behaviour**: node state, handlers, phrases. Example: the gossip protocol.
- **Programs**: a main file (`# package main`) whose `define main` builds a cluster, injects requests and faults, and stops. Example: "6 Cassandra nodes, crash one replica at 6s".
- **Looks**: observers that draw and animate. Example: a ring view that colours the Raft leader.

## 0.2 The pipeline

```
.synod sources ──► compiler ──► simulator ──► trace ──► scene builder ──► scene ──► player
                   (check)     (sim clock)   (facts)   (observers,         (shapes &   (JS canvas:
                                                        presentation clock) keyframes)  Mac app, site)
```

- The **simulator** runs on a deterministic simulated clock and writes a
  **trace**: an ordered list of facts (message sent, state changed,
  invariant broken…).
- The **scene builder** replays the trace, runs the observers of look
  packages on a **presentation clock**, and produces a **scene**: shapes
  with keyframes.
- The **player**, a JavaScript canvas library used by the Mac app and the website, only draws scenes. It never interprets Synod.

An LLM (Claude through its CLI, or any agent) only ever writes Synod source.

## 0.3 A first program

```synod
# package main
use github.com/synodlang/examples/tcp
use std/lesson
use std/sim

define main
  set client to (sim.Spawn tcp)
  set server to (sim.Spawn tcp)
  set server.tcp.State to "LISTEN"
  lesson.Note "A connection needs three messages before data can flow."
  deliver (message tcp.Open with Server server) to client
```

## 0.4 How to read this specification

- `spec/01` to `spec/04`: what the text looks like and what it means statically.
- `spec/05` to `spec/07`: what happens at run time.
- `spec/08` and `spec/09`: what can be observed and drawn.
- `spec/10` and `spec/11`: canonical form and errors.
- `spec/decisions.md`: why things are the way they are.

"Must", "must not" and "may" are normative. Code blocks marked `synod`
are canonical Synod. Blocks that start with `# package` are complete
files and must pass `synod check` together with the other files of the
same package (a main file is a package of its own); shorter blocks are fragments.
