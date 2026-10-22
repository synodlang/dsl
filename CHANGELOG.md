# Changelog

Changes to the Synod language specification, the std catalog and the
implementation guides. Each released version's section is the text of its
GitHub release page.

## Unreleased

## v0.2.10 — 2026-10-22

M10 (Claude integration, agent skills, `synod doc`, `synod explain`, `synod new`, `synod get` and phrase completion) is specified, and gaps G19 to G27 of Synod 0.2 are closed.

### Added

- `synod doc` (decisions 90 and 91, gaps G19 and G20): documents a package from its source, with doc comments, `--json` (`synod-doc/1`), `--catalog`, `--scope` and `--overlay`; every std `define` has a doc comment equal to its catalog Meaning, and Gives has one spelling per rule (`implementation/05-cli.md`, `implementation/04-formats.md`).
- Phrase completion in the Mac app (decision 92, gap G22), from `synod doc --json --scope`; decision 80 now says that `check` and `doc` take `--overlay`.
- `synod explain` (decision 93, gap G23) with the entry format and the `synod-explain/1` format, `synod new` (decision 94, gap G24) and `synod get` (decision 95, gap G25) with the download rule, the store, `synod.sum` (its exact line and `h1:` hash) and `vendor/` copies.
- The prompt bar of the Mac app (decision 96, gap G26): it runs the user's own Claude Code CLI after an opt-in, with an allowlisted environment and no credential handled by the app; `implementation/07-claude-integration.md` has the invocation.
- `agent-skills` as a Claude Code plugin with generated parts and pinned versions (decision 97, gap G27); the M10 acceptance (`implementation/10-roadmap.md`).

### Changed

- `std/README.md`: the three byte-for-byte blocks (`time`, `lesson`, `look`) gain their doc comments, the Meaning cells of `Place … below` and `Place … beside` are filled, and `same type as` becomes `same as` (the Meaning of `disk.Read` says "the type of `d`").
- `spec/06` §6.2 says that `synod get` copies (no hard links) and where a module comes from; §6.4 defines doc comments. `spec/11` §11.1 points to `synod explain`.
- `implementation/01-compiler.md` adds `internal/doc`, `internal/explain` and `internal/get` (gap G31); `05-cli.md` no longer says that `synod get` is not implemented (gap G32) and `synod version` lists `synod-doc/1` and `synod-explain/1`; `06-mac-app.md`, `08-agent-skills.md`, `09-site-and-org.md` (the sync workflow lives in `agent-skills`), `11-documentation.md` (the site build runs the native `synod doc --json` and `synod explain --json`, gap G30) and `00-architecture.md` follow.

## v0.2.9 — 2026-10-22

The M9 acceptance text for Break it says exactly what the check does.

### Changed

- `implementation/10-roadmap.md` M9 acceptance: the Cassandra hinted-handoff check types `6s` in the sheet's time field for the crash (the default time, 1ms after the current fact, changes the gossip course with seed 3 so no hint is stored) and uses the default time for the restart; still no source text is typed.

## v0.2.8 — 2026-10-22

A run-time error raised inside std is reported where the user can act on it.

### Changed

- A run-time error raised while std code runs is reported at the position in user code of the call that entered std, in `sim.Failed`, the `failed at` and `look failed at` lines and `--json`; the std position is kept as a note (decision 89; `spec/07` §7.8, `spec/11` §11.3, `implementation/04-formats.md`, `implementation/05-cli.md`).

## v0.2.7 — 2026-10-22

The editor and learning modes of the Mac app (M9) are specified, and three gaps
of Synod 0.2 are closed: where Predict gets its choices, which node or message
a shape is, and what Break it writes.

### Added

- `lesson.Ask [q: text] choosing [c: list of text]` in the std catalog (gap G16, decision 82): 2 to 6 distinct, non-empty choices, else `bad-argument`. `lesson.Asked` gains the field `Choices` (`{}` for a plain `Ask`) and a `pause` marker the key `choices`; `std/README.md` has the new `lesson.synod` byte for byte.
- `synod-scene/1` shape keys `node` and `msg`, written by the scene builder (gaps G15 and G17, decision 83), and the reader rules for `hitTest` in `04-formats.md` "Hit testing (readers)" (decision 84).
- `synod check --overlay <file>` and the `synod-overlay/1` format (decision 80), and lexical tokens for files that do not parse in `synod check --json --tokens` (decision 79).

### Changed

- Mac app (M9): `06-mac-app.md` specifies the editor (tokens from `synod check`, no lexer of its own, unsaved buffers by overlay, format on save, quick fixes applied only on request, go to definition), Predict choices, Break it (gap G18: `sim.Nodes`, `net.DropNext` at send time, Heal, verification of the effect and of broken rules), Compare (nodes matched by name, one timeline in simulated time instead of shape ids) and GIF export (schedule, palette, limits). Phrase completion moves to M10, with `synod doc`. Decisions 78 to 88; decision 74 now runs `synod check --json --tokens`.
- `10-roadmap.md`: G15 is resolved (the builder writes `node`, not `look.Card`), G16 to G18 are resolved, the M9 acceptance is concrete, and M10 notes that `agent-skills` is generated from the catalog as it stands. `05-cli.md`, `04-formats.md` (the Mac app applies a `help` edit when the user picks it), `03-scene-builder.md`, `spec/05`, `spec/08` and `00-architecture.md` follow.
- `spec/10`: a duration that is not a whole number of milliseconds is written in milliseconds with the fraction (`20.431ms`), as the compiler already does.

## v0.2.6 — 2026-10-22

The Mac app (M8) is specified: what it does, how it runs and exports, and
how it is released.

### Changed

- Mac app (M8): `implementation/06-mac-app.md` splits M8 from M9 (GIF export, the editor, Predict choices, Break it and Compare move to M9), says the timeline speed is the player rate and takes effect at once (the scene `--pace` is not exposed), describes the run and cache behaviour, the export frame schedule with one render path (`renderFrame`; the `OffscreenCanvas`/`renderToContext` alternative is removed), where Predict answers live, node choice from a list, and distribution. `10-roadmap.md` M8 gets its concrete acceptance (smoke at pace 0.9 to 1.1; a Raft export of N frames matching the stage player at PSNR 38 dB or more and closer to its own reference than to its neighbours) and M9 takes GIF; `05-cli.md` says the app may later use `--json-events` while M8 re-runs per change and both flags stay not implemented; `00-architecture.md` follows. Decisions 74 to 77.
- Gap G15 (open, for M9): clicking a node on the stage needs an additive optional shape key `node` in `synod-scene/1`, written by `look.Card` and `std/look/basic`, and a player `hitTest(x, y)`.

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
