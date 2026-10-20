# Roadmap for implementation

Build in this order. Each milestone ends with its acceptance checks
passing in CI. Do not start a milestone before the previous one passes.

## M0 — Organization skeleton

- Create the repositories listed in `00-architecture.md` under
  `github.com/synodlang`, each with a README, licence and CI stub.
- Push this `dsl` repository as `v0.1.0`.

**Accept:** every repository exists; `dsl` README renders.

## M1 — Lexer, parser, formatter (`compiler`)

- `synod fmt` and the `syntax`, `indent`, `non-canonical`, `bad-header` checks.

**Accept:** every example file round-trips through `fmt` unchanged; a
fuzz run of 10 minutes finds no crash; deliberately mis-formatted copies
of the examples format back to the originals.

## M2 — Loader, resolver, checker

- `synod.mod`, local packages, the bundled `std` declarations, phrase
  resolution, types, contexts, every code in `spec/11` §11.2.

**Accept:** `synod check` passes on all examples; a golden test per
error code reproduces it with the expected message.

## M3 — Simulator and std natives

- Event loop, nodes, network, disk, PRNG, invariants, trace writer;
  natives for `sim`, `net`, `time`, `random`, `lists`, `maps`, `math`,
  `text`, `disk`, `lesson`.

**Accept:** `tcp`, `gossip`, `paxos`, `raft`, `twopc` run with golden
traces; running twice gives byte-identical traces; Paxos's invariant
holds across seeds 1–200; breaking Paxos (removing the highest-prior
rule in `paxos.synod`) makes `sim.Broken` appear for some seed.

## M4 — Scene builder and looks

- `std/look`, `std/theme`, `std/look/basic`, observers, reactive draws,
  presentation clock, `scene.json`.

**Accept:** golden scenes for all examples; `raftlook` and `ringlook`
render; the scheduling rules of `spec/09` §9.6 are covered by tests.

## M5 — Mac app player

- Open a workspace, run, play scenes, timeline, inspector, video export.

**Accept:** every example plays at 120fps; exported 1080p video of Raft
matches the player frame for frame at the same pace.

## M6 — Editor and learning modes

- Editor with tokens and diagnostics, Predict, Break it, Compare.

**Accept:** Break it can reproduce the Cassandra hinted-handoff story by
clicks alone; Predict stops at each `lesson.Ask`.

## M7 — Claude integration and agent skills

- `agent-skills` with the generated catalog; the prompt bar.

**Accept:** "animate two-phase commit with a coordinator crash" produces
a checked program in one request, using the `twopc` package without
rewriting it.

## M8 — Remaining examples, homepage, `.github`

**Accept:** homepage live with recordings of every example.
