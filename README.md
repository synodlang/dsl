# Synod

Synod is a small, consistent language for describing distributed systems,
system designs and database internals as composable packages. A Synod
program is **simulated deterministically** and **animated** in a
3Blue1Brown-like style, so you can watch a protocol run, predict what
happens next, break it, and compare designs.

The name honours Leslie Lamport: the *Synod* protocol is the core
consensus algorithm of his paper "The Part-Time Parliament", known today
as basic Paxos.

```synod
# package gossip (excerpt)
node has
  set Heartbeat to 0
  set Known to {}

time.Every 1s
  set Heartbeat to (Heartbeat plus 1)
  net.Send (message digest with known Known) to (random.Pick peers)
```

## This repository

`dsl` is the source of truth for the language. Every other repository in
the [synodlang](https://github.com/synodlang) organization implements or
uses what is written here.

| Path | What it holds |
| --- | --- |
| `spec/` | The language specification, normative |
| `std/README.md` | The standard library catalog, normative |
| `implementation/` | How each repository is built: architecture, compiler, runtime, scene builder, file formats, CLI, Mac app, Claude integration, agent skills, roadmap |
| `examples/` | Sample programs in canonical Synod, to be moved to the `examples` repo |
| `assets/icon.svg` | The language icon |

## Reading order

For a person: `spec/00-introduction.md`, then the examples, then the rest of `spec/`.

For an implementing agent: read **everything in `spec/` and `std/`
first**, then `implementation/00-architecture.md` and
`implementation/10-roadmap.md`, and build milestone by milestone. When
the spec and an example disagree, the spec wins and the example is a
bug. When the spec is silent, choose the option that keeps "one way
only" true, and record the decision in `spec/decisions.md`.

## Design principles, in priority order

1. **Consistency over readability.** When the two conflict, consistency wins.
2. **One way only.** Every meaning has exactly one spelling.
3. **Canonical code.** The compiler accepts only canonical form; `synod fmt` produces it.
4. **A tiny language, everything else in packages.** The language is a fixed set of words; the standard library is ordinary packages plus a few native phrases.
5. **Readable as pseudocode** within the rules above.
6. **Writable offline.** No LLM is needed to write, check, run or animate Synod. Claude (or any agent) is an accelerator.
7. **Faithful simulation.** Behaviour is simulated, never narrated by an LLM. What you see is what the model does.

## Status

Specification v0.1, October 2026. The earlier design document in Claude
Docs is superseded by this repository.
