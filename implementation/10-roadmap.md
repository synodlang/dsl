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

## M8 — Mac app (`mac-app`)

- Open a workspace, run, play scenes with `player` in a WKWebView,
  timeline (speed is the player rate, instant), inspector (node chosen from
  a list), problems and a read-only source view, video export, and a minimal
  Predict (stops at each `lesson.Ask`, free-text answer).
- Run behaviour, export schedule and release rules: decisions 74 to 76
  (`06-mac-app.md`). Not in M8: GIF export, the editor, Predict choices,
  Break it, Compare (all M9).
- Release: `mac-app` v0.2.0 bundles `synod` v0.2.4, `player` v0.2.0 and the
  examples; it is ad-hoc signed and not notarized.

**Accept:**

- Smoke: every bundled example runs, loads in the player with no warnings or
  errors, and plays at 1x with a measured pace (player time over wall time)
  from 0.9 to 1.1.
- Export: Raft (seed 3, ordered) exported at 1080p, 60 fps, rate 1 has
  `N = ceil(D·60/1000) + 1` frames (8,880 for the golden scene) at
  presentation times `k/60`. A sample of frames (the first and last, one
  after each marker, 60 evenly spaced and 30 consecutive around the busiest
  moment) match `renderFrame(k·1000/60)` of the stage player at PSNR 38 dB or
  more (never below 34 dB), and each is closer to its own reference than to
  the references of frames `k-1` and `k+1`, so an off-by-one frame fails.

## M9 — Editor and learning modes

- Editor with tokens and diagnostics, format on save and quick fixes, GIF
  export, Predict choices, Break it, Compare, and choosing a node by clicking
  the stage. Phrase completion moves to M10: it needs `synod doc`, which is
  not implemented, and M10 needs `synod doc` for the agent-skills catalog.
- G15 (resolved, decisions 83 and 84): `synod-scene/1` gains the additive
  optional shape keys `node` and `msg`, written by the scene builder for every
  node attachment and `look.Flight` (not by `look.Card`, which is Synod code),
  and the player gains `hitTest(x, y)` and `nodeBounds()`.
- G16 (resolved, decision 82): `lesson.Ask` had no way to say what Predict
  offers. `std/lesson` gains `Ask [q: text] choosing [c: list of text]`,
  `lesson.Asked` the field `Choices`, and a `pause` marker the key `choices`
  (a gap of Synod 0.2, fixed in a patch like G14).
- G17 (resolved, decision 83): the `msg` key says which message a dot is.
- G18 (resolved, decision 85): Break it edits `define main` with
  `sim.Nodes` and `net.DropNext` at send time, and Heal is added. Decision 86
  covers verification, and decision 87 Compare.
- Editor, GIF and learning-mode rules: decisions 78 to 88 (`06-mac-app.md`,
  `04-formats.md`, `05-cli.md`). `synod check` gains `--overlay` and lexical
  tokens for files that do not parse (decisions 79 and 80).
- Release: compiler v0.2.5, std v0.2.1, examples v0.2.1, player v0.2.1,
  `mac-app` v0.2.1, in this order, after `dsl` v0.2.7 (decision 61).

**Accept:** (headless unless marked; the app has modes that write a JSON
report, and `scripts/acceptance-m9.sh` in `mac-app` runs them all)

- Editor: for every file of the bundled examples, the tokens map to valid
  UTF-16 ranges of the right length and every role has a style. In a copy of
  `gossip`, an unsaved edit that misspells `net.Send` as `net.Sned` gives an
  `unknown-phrase` underline over exactly `line:col` to `end` within 2s, with
  the disk file unchanged; its quick fix restores the text byte for byte. A
  buffer with an unsorted import saved equals the `synod fmt` output and starts
  a run within 2s. A file with an unbalanced `(` still has lexical tokens. The
  overlay check of `cassandra` takes under 300ms.
- Predict: every bundled program has at least one `pause` marker, each with 2
  to 6 choices; playing at rate 4 shows each choice list, and choosing the
  first saves `answer` and `choice` and playback reaches the end.
- GIF: `tcp` exported at 640 px, 20 fps, rate 1 decodes with the right frame
  count, every delay 0.05s, loop count 0, size 640×400, every pixel in the
  palette; sampled frames match `renderFrame(at_k)` at PSNR 30 dB or more (never
  below 28 dB) and each is closer to its own reference than to its neighbours;
  exporting twice gives identical bytes.
- Click to select: for every program, at 10 times, the center of every
  `nodeBounds` box hits its node or a message (at least 80% node hits, no
  mismatch), and a click dispatched on the stage selects that node.
- Break it: starting from the Cassandra demo without its scripted crash,
  clicks alone (Crash n5 at 6s, Restart at 18s) insert exactly two blocks,
  `sim.Crash (item 4 of sim.Nodes) # n5` and the matching `sim.Restart`, and
  the new trace has `sim.Crashed n5` at 6 to 7s, n2's `cassandra.PendingHints`
  at 1 or more after 12s, `sim.Restarted n5`, a `cassandra.Mutation` arriving at
  n5 afterwards and `PendingHints` back to 0, and n5 ends with
  `cassandra.Data {"user:42": "Grace"}`. Each step's check passes and no text
  is typed.
- Compare: `raft` with seeds 3 and 4 gives two runs with a monotone sync map;
  after `seekSim(T)` for 20 values of `T` each side is at its map value within
  0.5ms, and playing 2s at 1x keeps the right side within 17ms of the left
  side's mapped time; the node lists match by name.
- Regression: the M8 smoke and Raft export checks still pass.
- Manual, on an unlocked screen: typing feel, highlight colors and contrast,
  underlines and the lightbulb, ⌘-click navigation, real mouse clicks and
  drags, the Break it popovers, the Predict layout and keys, Compare
  smoothness at 120 Hz, GIF quality in Safari and Quick Look, and the
  Gatekeeper first launch of the release.

## M10 — Claude integration and agent skills

- `agent-skills` with the generated catalog; the prompt bar.
- `synod doc` (needed for the catalog) and, with it, phrase completion in the
  editor (moved from M9).
- `agent-skills` is still empty after M9, so the rule that it is regenerated
  when the std catalog changes could not apply yet: M10 generates it from the
  catalog as it stands, including `lesson.Ask … choosing`.

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
