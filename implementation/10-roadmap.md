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
  the stage. Phrase completion moves to M10, together with `synod doc`, which
  it needs and which M10 also needs for the agent-skills catalog.
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
  clicks alone insert exactly two blocks: Crash n5 after seeking to the last
  fact at or before 6s and typing `6s` in the sheet's time field (the default
  time, 1ms after that fact, shifts the gossip course with seed 3 so no hint is
  stored), then Restart n5 after seeking to the last fact at or before 18s with
  the sheet's default time. The blocks are
  `sim.Crash (item 4 of sim.Nodes) # n5` and the matching `sim.Restart`, and
  the new trace has `sim.Crashed n5` at 6 to 7s, n2's `cassandra.PendingHints`
  at 1 or more after 12s, `sim.Restarted n5`, a `cassandra.Mutation` arriving at
  n5 afterwards and `PendingHints` back to 0, and n5 ends with
  `cassandra.Data {"user:42": "Grace"}`. Each step's check passes and no source text
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

- `synod doc` (decisions 90 to 92), `synod explain` (93), `synod new` (94) and
  `synod get` (95) in the compiler; std doc comments equal the catalog
  (decision 91).
- Phrase completion in the editor (moved from M9), from `synod doc --json
  --scope` (decision 92).
- The prompt bar, which runs the user's own Claude Code CLI after an opt-in and
  handles no credential (decision 96), and `agent-skills` as a Claude Code
  plugin with generated parts (decision 97). `agent-skills` was empty after M9,
  so M10 generates it from the catalog as it stands, including
  `lesson.Ask … choosing`.
- MCP in M10 (G28, decisions 98 to 102): the editor server of the Mac app (a bundled stdio bridge over a Unix socket) and `synod mcp`; an in-app Messages-API chat stays out (no key handling).
- G33: unsaved Claude edits cannot run, because a run reads saved files (decision 81), so Claude saves first. G34: a new file must exist on disk to be checked (decision 80), so `create_file` creates it, and decision 99 names the two disk writes. G35: the external token is not stored, so a copied config stops working after a relaunch. G36: the Claude Code flag spellings, MCP tool names and protocol version strings are checked against the installed `claude` and modelcontextprotocol.io.
- G19 to G27 are resolved by decisions 90 to 97. G29: the live acceptance
  (11 below) needs the owner's Claude Code login, so CI cannot run it. G30: the
  site build runs the native `synod doc --json` and `synod explain --json`
  (`11-documentation.md`). G31: `synod get` lives in `internal/get`, and
  `internal/load` stays read-only (`01-compiler.md`). G32: `05-cli.md` no
  longer says that `synod get` is not implemented.
- Release: compiler v0.2.6, std v0.2.2, agent-skills v0.2.0, `mac-app` v0.2.2,
  in this order, after `dsl` v0.2.10; examples, player and the site are
  unchanged (decision 61).

**Accept:** (headless unless marked)

Compiler:

1. `synod doc` in the examples module lists the 13 std packages and the
   examples' library packages; `synod doc std/net` matches its golden.
2. Every row of `std/README.md` appears in `synod doc --catalog` with the same
   pattern (qualified), Gives, contexts and Meaning.
3. `synod doc --json --scope twopc/demo.synod` lists the imports `twopc`,
   `lesson`, `net` and `sim`, and `twopc.Coordinate [ps: list of twopc.Node]
   from [c: twopc.Node]` with 2 holes.
4. `synod explain` lists exactly the 38 codes of `spec/11`; every `synod
   explain <code>` exits 0, every entry's example gives only its code and its
   fix is clean, and `synod explain nope` exits 2.
5. `synod new` then `synod fmt --check .`, `synod check .` and `synod run
   <pkg>` all exit 0 in the new folder.
6. `synod get` passes the offline repository tests. Network check (CI only,
   allowed to fail when GitHub is unreachable): in a `synod new` module,
   `synod get github.com/synodlang/examples@v0.2.1` writes
   `vendor/github.com/synodlang/examples@v0.2.1/twopc/twopc.synod`, the
   `synod.sum` line has commit `0f15d8d8f41f6fcaa68d4b2bb921da63884dfafc`, and a
   second run with networking off prints `from cache` or `up to date`.

`agent-skills`:

7. `go run ./tools/sync --check` exits 0 with the pinned versions; every
   whole-file `synod` block and `reference/examples` pass `synod fmt --check`
   and `synod check`; the plugin version equals the CHANGELOG version; the
   release has the tarball and `SHA256SUMS`.
8. On screen: `claude plugin validate .` (when this Claude Code has it), then
   `/plugin marketplace add synodlang/agent-skills` and `/plugin install
   synod@synodlang` load the skill.

`mac-app` (`scripts/acceptance-m10.sh` runs `Synod --m10`):

9. Completion, in a temp copy of the examples:
   - for every `.synod` file, `doc --scope` decodes and its imports equal the
     qualifiers the compiler reports;
   - in `gossip/gossip.synod`, on a new line after the first `net.Send` line,
     typing `net.Se` makes `net.Send [m: msg] from [a: node] to [b: node]` the
     first item; accepting it makes the line `<indent>net.Send m from a to b`
     with placeholders at `m`, `a` and `b`, `m` selected; Tab selects `a`,
     typing `p` replaces it, Esc clears them;
   - in `raft/raft.synod`, inside a receiver phrase, typing `n.` offers exactly
     raft's receiver phrases and `Node` fields from the scope, and no std
     phrase;
   - in `twopc/demo.synod`, `twopc.Co` accepts to `twopc.Coordinate ps from c`;
   - after an unbalanced `(` (the overlay check fails), `net.` still offers
     net's phrases from the cached scope;
   - `doc --scope` of `cassandra/demo.synod` takes under 300 ms in the app, and
     computing the items for the largest example file under 16 ms.
10. Prompt bar, deterministic (`--claude-check --claude <fake>`), on a temp copy
    of a module with a dirty buffer:
    - Send is refused until Save All, and the save happens;
    - the fake's log shows the argv of `07-claude-integration.md` exactly (model
      `claude-opus-5-5`), the working directory equal to the module root, the
      environment keys inside the allowlist, `PATH` starting with the bundled
      `synod`'s folder, and the variable `SYNOD_SECRET_SENTINEL` (set in the app
      process) absent;
    - the progress lines equal the fixture's tool uses in order;
    - the fake's new file is opened, its program selected, and a run started
      within 2 s;
    - the banner shows the number of changed files; Revert restores every byte
      and removes the added file;
    - a second prompt carries `--resume <id from the fixture>`;
    - a slow request and Stop end the process within 3 s;
    - with the setting off, the fake's log does not exist (no launch);
    - a search of the app's sources finds no `SecItem`, `Keychain` or
      `ANTHROPIC`.
11. Prompt bar, live: needs the owner's Claude Code login and network, not CI
    (a locked screen is fine). `scripts/acceptance-m10-live.sh` runs the prompt
    "animate two-phase commit with a coordinator crash", one request and no
    follow-up, three times, each in a fresh module (`synod.mod` with `module
    example.com/acc`, `synod 0.2` and `require github.com/synodlang/examples
    v0.2.1`, a `.gitignore`, and the bundled `synod get`). A run passes when
    all of these hold:
    - the result is not an error and `synod check .` exits 0;
    - exactly one new main file, and `synod run <its folder>` exits 0;
    - the trace header `packages` contains
      `github.com/synodlang/examples/twopc`;
    - the trace has a `sim.Crashed` whose node's last `sim.Added` or
      `sim.Changed` state has a non-empty `twopc.Participants` (the
      coordinator);
    - no new file declares `# package twopc` or a `Prepare has`, `Vote has` or
      `Decision has` record;
    - the h1 of `vendor/…@v0.2.1` is unchanged.

    All 3 of 3 must pass. A failure is fixed in `SKILL.md`, then `agent-skills`
    and the app's pin are re-released, never by retrying. The report goes into
    the `mac-app` release pull request. The transcript must show
    `mcp__synod-editor__` calls and no Write or Edit, all conditions hold after
    `save_files`, and the `protocolVersion` Claude Code sent is recorded. The
    owner also checks an external session (Settings toggle on, Copy MCP config,
    an edit asked in Terminal appears unsaved in the app and ⌘Z undoes it,
    Disconnect makes the same request fail), and `claude mcp add synod --
    synod mcp` with the answers to "explain unknown-phrase" and "summarize a run
    of raft" matching the CLI and no new `out/`.
12. Regression: `--m9 --quick`, the M8 smoke and acceptance scripts still pass.
13. `synod mcp` (compiler CI): a scripted session piped into `synod mcp` in the
    examples module gives, for each tool, text equal to the CLI output, and
    `run_summary` leaves `git status --porcelain` empty.
14. `--mcp-check <examples copy>` in the app (a locked screen is fine), with the
    built bridge and a scripted MCP client:
    - `tools/list` equals the golden;
    - after dirtying `gossip/gossip.synod`, `read_buffer` returns the unsaved
      text;
    - `apply_edits` (misspelling `net.Send` as `net.Sned`) makes a fresh
      `get_diagnostics` report `unknown-phrase` within 2 s while the disk is
      unchanged, and one undo restores the bytes;
    - a stale version gives `version-mismatch`;
    - `save_files` makes the disk equal the `synod fmt` output and starts a run
      within 2 s;
    - `create_file` of `extra/demo2.synod` lands formatted and a second call
      gives `exists`;
    - `../x` gives `outside-module` and the bundled examples give `read-only`;
    - a wrong token is refused;
    - while running, `lsof -nP -a -p <pid> -iTCP -sTCP:LISTEN` is empty, the
      socket is 0600 in a 0700 folder, and after the session the socket, the
      folder and the config file are gone;
    - with external access off a bridge connection is refused; with it on, a
      bridge with `--workspace` works; after Disconnect the old token is
      refused.
15. `--claude-check` (extends 10) with the fake Claude: the argv has
    `--mcp-config`, `--strict-mcp-config` and the exact allowed and disallowed
    lists of `07-claude-integration.md`; the config file is 0600 during the run
    and gone afterwards; the scripted edits appear as "Claude Edit" undo groups;
    the progress lines show the MCP tool names; Revert restores the snapshot; no
    Write or Edit tool appears.

Manual, on an unlocked screen: the completion popup (look, caret anchor,
keyboard feel, placeholder drawing); the prompt bar (layout, live progress, Stop,
the banner, Show changes, Revert); the Settings, Claude opt-in sheet and the
"claude not found" and "not logged in" messages; one real prompt from the bar,
end to end; the Gatekeeper first launch of the release.

## M11 — Remaining examples, site, documentation, `.github`

- The site and its documentation as in `09-site-and-org.md` and
  `11-documentation.md`: tutorial, reference, library, guides, errors.
- Tutorial chapters may be drafted from M2 on, since their code is
  checked by `synod check` from then; this milestone finishes them.

**Accept:** synodlang.github.io live; every example page and every
tutorial step is editable and re-runs in the browser; the tutorial
coverage check passes; every diagnostic code has an Errors page; no
recordings.
