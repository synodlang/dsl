# Synod

The specification of Synod, a small, consistent language for describing distributed systems, with the standard library catalog and the guides for building it.

[![ci](https://github.com/synodlang/dsl/actions/workflows/ci.yml/badge.svg)](https://github.com/synodlang/dsl/actions/workflows/ci.yml)
[![release](https://img.shields.io/github/v/release/synodlang/dsl)](https://github.com/synodlang/dsl/releases/latest)
[![licence](https://img.shields.io/badge/licence-CC%20BY%204.0-lightgrey)](LICENSE)

Synod is a small, consistent language for describing distributed systems,
system designs and database internals as composable packages. A Synod
program is **simulated deterministically** and **animated** in a
3Blue1Brown-like style, so you can watch a protocol run, predict what
happens next, break it, and compare designs.

## Why Synod

Synod is named after the Synod protocol, the core consensus algorithm in Leslie Lamport's paper "The Part-Time Parliament" (ACM Transactions on Computer Systems, 1998; first submitted in 1990). The paper tells it as the story of the parliament of the ancient Greek island of Paxos, whose part-time legislators could agree on decrees although they came and went; today the algorithm is known as Paxos. Lamport later explained it plainly in "Paxos Made Simple" (2001), and in 2013 he received the ACM A.M. Turing Award for fundamental contributions to the theory and practice of distributed and concurrent systems. His "Time, Clocks, and the Ordering of Events in a Distributed System" (1978) and his TLA+ specification language shaped how we reason about such systems. We are grateful for his work, and we hope Synod honours it by making these ideas easier to watch and understand. This repository is the specification of the language that carries his protocol's name.

```synod
# from package gossip
Node has
  set Heartbeat to 0
  set Known to {}
  set peers to {}

define [n: Node] start
  time.Every 1s
    set n.Heartbeat to (n.Heartbeat plus 1)
    net.Send (message digest with known n.Known) from n to (random.Pick n.peers)
```

## A taste of Synod

This is the whole TCP handshake, from the
[examples](https://github.com/synodlang/examples) repository. Each `when`
line says which node reacts to which message. Every line reads as a
sentence.

```synod
# package tcp
use std/net
use std/sim

Open has
  set Server to sim.Nobody

Syn has
  set Seq to 0

SynAck has
  set Seq to 0
  set AckNo to 0

Ack has
  set AckNo to 0

Node has
  set State to "CLOSED"
  set seq to 0
  set peerSeq to 0

# Client: the application asks to open a connection
when [n: Node] (sim.Delivered Open)
  set n.seq to 100
  set n.State to "SYN-SENT"
  net.Send (message Syn with Seq n.seq) from n to Open.Server

# Server: answer a SYN with SYN-ACK
when [n: Node] (net.Received Syn from client)
  set n.peerSeq to Syn.Seq
  set n.seq to 300
  set n.State to "SYN-RECEIVED"
  net.Send (message SynAck with Seq n.seq, AckNo (n.peerSeq plus 1)) from n to client

# Client: a SYN-ACK that acknowledges our SYN completes our side
when [n: Node] (net.Received SynAck from server)
  if (SynAck.AckNo is (n.seq plus 1))
    set n.peerSeq to SynAck.Seq
    set n.State to "ESTABLISHED"
    net.Send (message Ack with AckNo (n.peerSeq plus 1)) from n to server

# Server: the final ACK completes the handshake
when [n: Node] (net.Received Ack from client)
  if (Ack.AckNo is (n.seq plus 1))
    set n.State to "ESTABLISHED"
```

A program (`# package main`) builds a cluster, sends a request and stops:

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
  after 5ms
    lesson.Ask "What does the server send back after a SYN?"
  after 100ms
    lesson.Note "Both ends are ESTABLISHED: data can now flow either way."
  sim.StopAt 200ms
```

Run it with the [compiler](https://github.com/synodlang/compiler):
`synod run ./tcp`. It prints the same trace every time.

## Where it fits

`dsl` is the source of truth for the language. Every other repository in
the [synodlang](https://github.com/synodlang) organization implements or
uses what is written here.

```
.synod sources --> compiler --> simulator --> trace --> scene builder --> scene --> player
                   (check)      (sim clock)   (facts)   (observers)       (shapes)   (canvas)
```

| Repository | What it is | Status |
| --- | --- | --- |
| `dsl` (this one) | The specification, the std catalog, the implementation guides | v0.2 |
| [compiler](https://github.com/synodlang/compiler) | The `synod` command: `fmt`, `check`, `run` | v0.2.2 implements Synod 0.2 |
| [std](https://github.com/synodlang/std) | The standard library packages | v0.2 |
| [examples](https://github.com/synodlang/examples) | Sample programs; the acceptance tests | v0.2 |
| `player`, `mac-app`, `agent-skills`, the site | Animation player, Mac app, agent skills, website | Later milestones (M5 onwards in the roadmap) |

The compiler checks and runs programs today. The scene builder, which turns
a trace into an animation, and everything after it are still to come.

## What is in this repository

| Path | What it holds |
| --- | --- |
| `spec/` | The language specification, normative |
| `std/README.md` | The standard library catalog, normative |
| `implementation/` | How each repository is built: architecture, compiler, runtime, scene builder, file formats, CLI, Mac app, Claude integration, agent skills, site, roadmap, documentation |
| `assets/icon.svg` | The language icon |
| `CHANGELOG.md` | What changed in each release of the spec, the catalog and the guides |

## The specification

Read the chapters in order. "Must", "must not" and "may" are normative.
Code blocks marked `synod` are canonical Synod; blocks that start with
`# package` are complete files and must pass `synod check`.

| Chapter | What it covers |
| --- | --- |
| [0. Introduction](spec/00-introduction.md) | What Synod describes, the pipeline, a first program, how to read the spec |
| [1. Lexical structure](spec/01-lexical.md) | Source files, lines and indentation, comments, words, numbers, durations, text, colors, paths, reserved words |
| [2. Grammar](spec/02-grammar.md) | The two-stage parse: the structure grammar, then the language word grammar |
| [3. Phrases](spec/03-phrases.md) | Phrases: sentence patterns of words and typed holes, defined once |
| [4. Types and values](spec/04-types-and-values.md) | Value types, structural types, records, lists and maps, operators, type inference |
| [5. Nodes, state and contexts](spec/05-state-and-contexts.md) | Node types, node state, receivers, the four contexts, events, invariants |
| [6. Packages, modules and programs](spec/06-packages-and-programs.md) | Packages, modules and `synod.mod`, imports, exports, programs and `define main` |
| [7. Execution model](spec/07-execution-model.md) | The event loop, timers, crash and restart, the network, determinism |
| [8. Trace events](spec/08-trace-events.md) | The facts a run produces |
| [9. The visual layer](spec/09-visual-layer.md) | Shapes, motion, observers, the presentation clock |
| [10. Canonical form](spec/10-canonical-form.md) | The one accepted spelling of every construct |
| [11. Errors](spec/11-errors.md) | Diagnostics: the short and rich forms, every compile-time and run-time code |
| [decisions](spec/decisions.md) | Why the language is the way it is |

### Design decisions

`spec/decisions.md` is a numbered table of what was decided and why. Add an
entry whenever an implementation has to decide something the spec did not.
The rules for the table:

- Every decision holds on its own scope. No decision overrides another.
- The one place where a program may pick a second spelling is `use ... as`,
  and only when two names would collide (decision 7).
- A design that cannot satisfy all the principles below changes; no
  principle is traded against another (decision 1).

## The standard library catalog

[`std/README.md`](std/README.md) lists every phrase of the standard
library, normative: its pattern, what it gives back, where it may be used,
and whether it is native (N, implemented by the compiler) or written in
Synod (S). The packages are `sim`, `net`, `time`, `random`, `lists`,
`maps`, `math`, `text`, `disk`, `lesson`, `theme` and `look`. The sources
are in the [std](https://github.com/synodlang/std) repository, and the
compiler embeds a copy of them; its tests check that the catalog, the
sources and the copy agree.

## Implementation guides

The guides in `implementation/` say how each repository is built. They are
written for people and for implementing agents.

| Guide | About |
| --- | --- |
| [00 Architecture](implementation/00-architecture.md) | The repositories, versioning, data flow, performance targets |
| [01 Compiler](implementation/01-compiler.md) | The packages of the compiler |
| [02 Runtime](implementation/02-runtime.md) | The simulator |
| [03 Scene builder](implementation/03-scene-builder.md) | From trace to scene |
| [04 Formats](implementation/04-formats.md) | `synod.mod`, `trace.jsonl`, `check.json`, `scene.json` |
| [05 CLI](implementation/05-cli.md) | The `synod` command, exit codes, distribution |
| [06 Mac app](implementation/06-mac-app.md) | The editor and player host |
| [07 Claude integration](implementation/07-claude-integration.md) | Writing Synod with Claude |
| [08 Agent skills](implementation/08-agent-skills.md) | Skills for coding agents |
| [09 Site and organization](implementation/09-site-and-org.md) | The website and the organization repositories |
| [10 Roadmap](implementation/10-roadmap.md) | Milestones M0 to M11 and their acceptance checks |
| [11 Documentation](implementation/11-documentation.md) | The documentation site |

## Reading order

For a person: `spec/00-introduction.md`, then the [examples](https://github.com/synodlang/examples), then the rest of `spec/`.

For an implementing agent: read **everything in `spec/` and `std/`
first**, then `implementation/00-architecture.md` and
`implementation/10-roadmap.md`, and build milestone by milestone. When
the spec and an example disagree, the spec is normative and the
example is a bug. When the spec is silent, choose the option that keeps "one way
only" true, and record the decision in `spec/decisions.md`.

## Design principles

These principles hold together. When a design cannot satisfy all of them,
the design changes; no principle is traded against another.

- **Consistency.** The same meaning looks the same everywhere.
- **One way only.** Every meaning has exactly one spelling.
- **Canonical code.** The compiler accepts only canonical form; `synod fmt` produces it.
- **A tiny language, everything else in packages.** The language is a fixed set of words; the standard library is ordinary packages plus a few native phrases.
- **Every line reads as pseudocode.**
- **Writable offline.** No LLM is needed to write, check, run or animate Synod. Claude (or any agent) is an accelerator.
- **Faithful simulation.** Behaviour is simulated, never narrated by an LLM. What you see is what the model does.

## Versioning

Every repository shares MAJOR.MINOR with the language version (decision 61).
Language 0.2 means every repository releases `v0.2.x`, and each repository
numbers its own PATCH.

- A `dsl` patch is a clarification or a spec bug fix. A patch elsewhere is a
  bug fix, or a feature that implements more of the same language version.
- A new language feature raises MINOR. The spec changes first, here, and each
  repository moves to the new MINOR when it implements it.
- Release notes state which Synod version is implemented and what changed.
  The notes of a `dsl` release are its section of [CHANGELOG.md](CHANGELOG.md).

## Propose a change

A change to the language starts as a pull request to this repository. It
must:

1. Update the spec chapters it touches, and the std catalog if a phrase
   changes. Code blocks stay canonical.
2. Add or amend an entry in `spec/decisions.md`. The table stays consistent:
   no entry overrides another.
3. Come with matching pull requests to
   [examples](https://github.com/synodlang/examples) and to the tutorial
   (planned in `tutorial/`, see `implementation/11-documentation.md`).
4. Add a section to `CHANGELOG.md` under "Unreleased".

If you are unsure, open a language proposal issue first. Files must use LF
line endings, have no trailing spaces and end with a newline; CI checks this
for every Markdown, `.synod` and `.mod` file.

The organization's [contributing guide](https://github.com/synodlang/.github/blob/main/CONTRIBUTING.md)
has the rest.

## Status

Specification v0.2, October 2026 (v0.1.0 was the first draft, before review). The earlier design document in Claude
Docs is superseded by this repository.

Licensed under [CC BY 4.0](LICENSE).
