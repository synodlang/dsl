# Changelog

Changes to the Synod language specification, the std catalog and the
implementation guides. Each released version's section is the text of its
GitHub release page.

## Unreleased

## v0.2.5 — 2026-10-22

How a player draws a scene, written down precisely enough that every player
draws the same frames.

### Changed

- Player rules (M7): `implementation/04-formats.md` gains "Drawing a scene (readers)" (lifetime, per-property tracks and defaults, smootherstep and OKLab, groups, geometry, lines, glow, order, letterbox), corrects "First key" for lines (opacity and stroke, no position) and says the `trace` map lets a host show simulated time by `seq`. `spec/09` §9.3: glow is the only effect, readers draw unknown effects as `none`; §9.5: in 0.2 a label is sans 16 px weight 500. `06-mac-app.md`, `09-site-and-org.md`, `11-documentation.md`, `00-architecture.md` (player distribution) and the M7 acceptance in `10-roadmap.md` follow. Decisions 72 and 73.

## v0.2.4 — 2026-10-22

The WebAssembly build is specified, and the call-depth limit is lowered so
that programs behave the same in a browser as on the command line.

### Changed

- The call-depth limit (`run-limit`, `spec/07` and `spec/11`) is 500 nested calls on every target, down from 10000, so browsers never overflow their stack running the WebAssembly build and outputs stay byte-identical everywhere.
- WebAssembly build (M6): `implementation/01-compiler.md` specifies the JS API (`check`, `run`, `format`, `version`; a global `synod`, synchronous, never throws, `files` rules, other modules under `vendor/`), the `internal/pipeline` and `internal/jsapi` packages, the release assets and the cross-target checks; `scene(files, ...)` is gone. `10-roadmap.md` M6 and its acceptance, `05-cli.md` (wasm assets, `check`/`fmt` skip `vendor` folders), `09-site-and-org.md`, `11-documentation.md` and `04-formats.md` follow. Decisions 70 and 71.

## v0.2.3 — 2026-10-22

Completes Synod 0.2: the visual layer is fully specified and every look in
the standard library is written in Synod (gap G14 closed).

### Changed

- Scene builder and looks (M5): `spec/09` states drawing, attachments,
  replacement, tombstones, groups, `Place` as a constraint, layout positions
  and pinning (§9.2, §9.4), `Tween` of `"x"`/`"y"` with a shape, the morph
  rule, tween supersede and `look.Flight` (§9.3), color names (§9.5), and the
  presentation clock in integer milliseconds with pace in thousandths, the
  scale formula, the constants, the shapes rule 4 waits for, first slot 0,
  proportional mode, look `after` timers and the 2 s tail (§9.6), and the
  default look ("uses std/look", §9.7). Decisions 63 to 69. `spec/05`: a look
  reads only exported fields, record fields included, and sees the state
  after its fact. `spec/08`: `net.Id` and `net.Type`. `spec/11` §11.3: look
  faults are never in the trace and stop the scene (decision 68).
- G14 resolved: new natives `look.Flight`, `net.Id`, `net.Type`,
  `sim.StateRows` and the unexported `look.delete`; `Travel`, `Remove` and
  `std/look/basic` become ordinary Synod.
- `implementation/03-scene-builder.md` rewritten; `04-formats.md` gives the
  exact `scene.json` layout (key order, first key, `end`, the `failed` marker,
  numbers rounded to 0.01, theme order, color names; readers ignore unknown
  marker kinds); `05-cli.md`: `synod run` always writes the trace and the
  scene, `--scene`, `--pace`, `--mode`, the scene status line, the `look
  failed` line and exit 1.

- Rich diagnostics (M4): `spec/11` §11.1 defines the rich form beside the short form, the no-position form `path: error[code]: message`, and the text style of labels, help and notes. `check.json` gains `end`, and `label`, `labels`, `help` (with edits) and `notes`; `implementation/04-formats.md` states that within a major version keys are only added and readers ignore unknown keys. `implementation/05-cli.md` documents `--style` and `--color` and the terminal and `NO_COLOR` rules. The agent and Mac app guides and the M4 roadmap entry follow.

## v0.2.2 — 2026-10-21

The simulator's rules, written down while building `synod run`, and a better
`math.Hash`. Programs that use `math.Hash` get different (better spread)
values than before; nothing else changes the meaning of a valid program.

### Changed

- M3 simulator rules stated: seed precedence (`--seed`, then `sim.Seed`, then
  1), step and time-limit rules, `sim.StopAt` boundary, evaluation order, RNG
  consumption, restart, `after` in shared code, node key per trace event,
  `sim.Failed` last with no `sim.Ended`, module-relative trace paths, run-time
  codes `bad-argument`, `number-range`, `run-limit` (decision 62).
- `math.Hash` is FNV-1a 64 mixed with the MurmurHash3 64-bit finalizer, then
  `(h >> 11) / 2^53`, so that similar texts (`user1`, `user2`) spread over
  [0, 1) (gap G11 refined).
- Time `×` and `÷` a number round down; `-0` is `0`.
- Downloads: shared store `~/.synod/cache`, `vendor/` stays the only place the
  compiler reads (decision 60, refined).
- Versioning: every repository shares MAJOR.MINOR with the language version
  (decision 61).
- Roadmap: new milestone M4 "Rich diagnostics"; later milestones renumbered.

## v0.2.1 — 2026-10-21

Clarifications and tightened rules found while building the compiler's
checker. The published examples stay valid; a program that relied on a case
v0.2.0 left open (such as `result` inside an `after` block) now gets an error.

### Changed

- A sentence's phrase is chosen by its shape: word count, literal words and
  blocks. Hole types never select between phrases; a wrong argument type is
  a `type-mismatch` on that argument (decision 52).
- `result` is allowed only while its phrase is still running, so not inside
  an `after` block or a block argument (decision 53). Setting a captured local
  inside such a block is a `bad-place` (decision 54).
- Locals are block scoped (decision 55). Look code reads only exported state
  (decision 56). The std catalog's Context column is enforced at every call
  (decision 57). Draw names are shape locals (decision 58). A place's type is
  the join of its assignments (decision 59).
- `use … as` also applies when an import's last segment equals the importing
  package's own name (decision 7, clarified).
- Downloaded modules live in the module's `vendor/` folder, ignored by git;
  there is no global cache (decision 60). Package discovery skips `vendor`,
  `testdata` and folders starting with `.` or `_`.
- Canonical form: the layout of headers, imports, blank lines and comments;
  `+1`, `.5` and `x-1` are syntax errors; CRLF and a byte order mark are
  fixed by `synod fmt`; `not (…)` is rewritten both ways, only around a
  single comparison; `synod fmt` never inserts parentheses (decisions 43–51).
- spec/11 lists, for every error code, every case that reports it.
- std catalog: each `# package` block equals a std source file, and the
  relations between arguments (item, key and default types) are stated.
- Implementation guides: binary distribution (macOS, Linux, Windows; Homebrew,
  Scoop, install.sh, go install), the install page, the documentation and
  tutorial plan, and the M1/M2 compiler design.

## v0.2.0 — 2026-10-21

The reviewed specification: explicit receivers, entry definitions
(`define main`, `define [n: Node] start`), `.` for named members, typed
collections, node effects that name their node, main files next to their
library, and a decisions table in which no rule overrides another.
