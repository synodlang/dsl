# Roadmap for implementation

Build in this order. Each milestone ends with its acceptance checks
passing in CI. Do not start a milestone before the previous one passes.

## M0 — Organization skeleton

- Create the repositories listed in `00-architecture.md` (including
  `player` and the site) under `github.com/synodlang`, each with a README, licence and CI stub.
- Push this `dsl` repository as `v0.2.0` (`v0.1.0` is the first draft, before review).

**Accept:** every repository exists; `dsl` README renders.

## M1 — Lexer, parser, formatter (`compiler`)

- `synod fmt` and the `syntax`, `indent`, `non-canonical`, `bad-header` checks.

- Release: `compiler` v0.1.0 shipped `synod fmt`, `synod check` and
  `synod version` (distribution in `05-cli.md`), before the versioning rule;
  v0.2.0 (M2, full `synod check`) is the first release under it. Later milestones release
  new patch versions of the same language version (`run` arrives with M3);
  see "Versioning" in `00-architecture.md`.

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
traces (the new examples); running twice gives byte-identical traces; Paxos's invariant
holds across seeds 1–200; breaking Paxos (removing the highest-prior
rule in `paxos.synod`) makes `sim.Broken` appear for some seed.

## M4 — Rich diagnostics

- Human rendering of diagnostics and run-time errors, Rust-style: the
  source line, `^^^` under the exact span, labelled secondary spans
  (`defined here`), and a `help:` with the fix shown as a diff. Color only
  on terminals, and never when `NO_COLOR` is set.
- The first line stays the exact one-line message (`spec/11` §11.1) that
  agents read. `--json` gains span ends and labels. Run-time errors
  (`sim.Failed`) render the same way. Rich output is the default only on a
  terminal (`--style rich|short`, `--color auto|always|never`; see
  `implementation/05-cli.md`).
- Golden policy: each `testdata/check` case has `expect.txt` (short form,
  unchanged), `expect-rich.txt` (rich, no color) and `expect.json`; each
  `testdata/run/errors` case has `expect-rich.txt`.

**Accept:** a golden rendering per diagnostic code and per run-time code;
the first line of each equals the one-line form; no escape codes with
`NO_COLOR` or when output is not a terminal; `--json` spans and labels
match the rendering.

## M5 — Scene builder and looks

- `std/look`, `std/theme`, `std/look/basic`, observers, reactive draws,
  presentation clock, `scene.json`.

- G14 (resolved, decisions 63 to 69): the natives `look.Flight … for m`,
  `net.Id`, `net.Type`, `sim.StateRows` and the unexported `look.delete`,
  and `look.Tween` of `"x"` and `"y"` accepting a shape. `Travel`, `Remove`
  and `std/look/basic` are ordinary Synod. G14 completes Synod 0.2 (`dsl`,
  std and compiler v0.2.3).
- `synod run` always writes `out/trace.jsonl` and `out/scene.json`; a look
  fault stops the scene, not the simulation (decision 68).
- Scene builder rules and `scene.json` layout: `03-scene-builder.md`,
  `04-formats.md`. Looks have golden scenes for all nine examples at their
  own seeds (raft with `raftlook` over `basic`, ring with `ringlook` only,
  the rest with `basic`) and scheduling tests per rule.

**Accept:** golden scenes for all examples; `raftlook` and `ringlook`
render; the scheduling rules of `spec/09` §9.6 are covered by tests.

## M6 — WebAssembly build (`compiler`)

- `GOOS=js GOARCH=wasm` build of `cmd/synod-wasm` with the JS API
  `check` / `run` / `format` / `version` (`01-compiler.md`).

**Accept:** `synod.wasm` produces traces, scenes, `check.json` and status
text byte-identical to native for every example; release assets published.

## M7 — Player (`player`)

- TypeScript canvas library that plays `scene.json`: keyframes,
  smootherstep, OKLab colors, markers, `renderFrame(at)` for export, drawing
  by the reader rules of `04-formats.md`.

**Accept:** the display-list goldens of every golden scene match exactly in
CI; WebKit pixel goldens match within tolerance in a pinned container; in
Safari on Apple silicon at 120 Hz, JS frame work has p99 ≤ 4 ms for every
golden scene and for a 50-node, 200-message stress scene.

## M8 — Mac app

- Open a workspace, run, play scenes with `player` in a WKWebView,
  timeline, inspector, video export.

**Accept:** every example plays; exported 1080p video of Raft matches
the player frame for frame at the same pace.

## M9 — Editor and learning modes

- Editor with tokens and diagnostics, Predict, Break it, Compare.

**Accept:** Break it can reproduce the Cassandra hinted-handoff story by
clicks alone; Predict stops at each `lesson.Ask`.

## M10 — Claude integration and agent skills

- `agent-skills` with the generated catalog; the prompt bar.

**Accept:** "animate two-phase commit with a coordinator crash" produces
a checked program in one request, using the `twopc` package without
rewriting it.

## M11 — Remaining examples, site, documentation, `.github`

- The site and its documentation as in `09-site-and-org.md` and
  `11-documentation.md`: tutorial, reference, library, guides, errors.
- Tutorial chapters may be drafted from M2 on, since their code is
  checked by `synod check` from then; this milestone finishes them.

**Accept:** synodlang.github.io live; every example page and every
tutorial step is editable and re-runs in the browser; the tutorial
coverage check passes; every diagnostic code has an Errors page; no
recordings.
