# Changelog

Changes to the Synod language specification, the std catalog and the
implementation guides. Each released version's section is the text of its
GitHub release page.

## Unreleased

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
