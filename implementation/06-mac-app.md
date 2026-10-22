# Mac app (repo `mac-app`)

SwiftUI, macOS 15+, Apple silicon first (the bundled `synod` is the
universal binary). It bundles the `synod` binary and treats it as its
engine. Offline-first: the app needs no network; only the prompt bar, which
runs the user's own Claude Code, uses one.

**Milestones.** M8 ships the workspace, runner, player, timeline,
inspector, problems and a read-only source view, video export and a
minimal Predict. M9 adds the editor, GIF export, Predict choices, Break it
and Compare. M10 adds phrase completion and the prompt bar. Each component below says which milestone it belongs to.

## Main window

```
┌───────────────┬──────────────────────────────────────────────┐
│ Sidebar       │ Stage (player)                               │
│  Programs     │                                              │
│  Packages     │     n1 ●───────● n2        "Who becomes…?"   │
│  Examples     │        ╲  •   ╱                              │
│               │         ● n3                                 │
│               ├──────────────────────────────────────────────┤
│               │ Timeline: ▶ ⏸  ◀ step ▶   speed 1×  ordered ▾ │
│               ├──────────────────────────────────────────────┤
│               │ Editor (.synod) │ Inspector (state, trace)   │
└───────────────┴──────────────────────────────────────────────┘
```

M8 shows the source read-only (opened from Problems or the sidebar); the
editor is M9; the prompt bar (M10) sits above the editor, see "Claude" below.

## Components

- **Editor (M9; M8 has a read-only source view).** TextKit 2
  (`NSTextView` with a text layout manager). The app has no lexer: syntax
  highlighting, go-to-definition and diagnostics come only from
  `synod check --json --tokens` (decision 78).
  - *Tokens.* Every run brings the tokens of the files of its program (the
    runner below). A file outside the current program gets
    `synod check --json --tokens <its folder>` (debounced, cancellable).
    Columns are code points; the app converts them to UTF-16 for `NSString`.
    Between two checks the token ranges move with the edits: tokens after an
    edit shift, tokens it overlaps are dropped. A file that does not parse
    still has lexical tokens (decision 79), so highlighting survives typing.
  - *Unsaved buffers.* 300ms after the last keystroke the app writes the dirty
    buffers to its cache folder
    (`~/Library/Caches/io.github.synodlang.synod/overlay/`), writes an
    `overlay.json` and runs `synod check --json --tokens --overlay` from the
    module root, cancelling a check still in flight (decision 80). A result is
    applied only if the buffer has not changed since the snapshot. Unsaved
    text is never run, and nothing is written into the workspace.
  - *Colors.* Paper ground and Ink text; by role: `comment` Quiet italic,
    `language` Accent, `phrase` Ink semibold, `name` Ink, `field` Good,
    `type` Warning, `value` Critical.
  - *Diagnostics.* An underline from `line:col` to `end` (Critical for an error,
    Warning for a warning; an empty span marks one character), a dotted Quiet
    underline with a tooltip for each secondary `label` (labels in other files
    are listed in Problems), gutter marks, and the message, hint and label on
    hover. A quick fix is a `help[].edits`; it is applied only when the user
    picks it (⌘. or the lightbulb), to its own file's buffer, in descending
    position order, as one undo group. A help across files is text only
    (decision 81).
  - *Save.* ⌘S writes the file atomically and runs
    `synod fmt <file> --style short --color never` from the module root. On
    exit 0 the text is reloaded as one undoable "Format" change with the
    selection kept by line and column; on exit 1 the file stays as written and
    the diagnostics show. The file watcher then starts the run, so every run
    reads a saved, canonical file.
  - *Go to definition.* ⌘-click a token with `def`; a `def.file` of the form
    `<module>@<version>/path` opens `vendor/<module>@<version>/path`.
  - *Outside changes.* A changed file reloads when its buffer is clean; with a
    dirty buffer a banner offers "Reload" or "Keep mine". The bundled examples
    are read-only, with a "Copy Examples to Folder…" banner.
  - *Phrase completion (M10, decision 92).* Items come only from
    `synod doc --json --scope <file>` (`04-formats.md`), so the app guesses
    no qualifier and has no lexer: the word being typed is the run of letters,
    digits and `.` before the caret, not a token.
    - *Scope.* After any check of the file's folder (a run, an overlay check or
      a folder check) with no `error` diagnostic, the app runs `synod doc
      --json --scope <relative path> [--overlay <the same overlay.json>]` from
      the module root, debounced 150ms, cancellable, and caches the result per
      file. On exit 1 it keeps the previous scope, so completion works while a
      line is half typed. Completion is off, with a note in Settings, when
      `synod version` lacks `synod-doc/1`.
    - *Context.* The prefix is the run of `[A-Za-z0-9.]` code points directly
      before the caret, and the character before the run is the start of the
      line, a space, `(`, `{` or `,`. It is split at the last `.`:

      | Typed | Items |
      | --- | --- |
      | no dot | own package phrases (bare heads) and every import's exported, non-receiver phrases (qualified heads) whose head, or `qual.head`, starts with the partial |
      | `q.` with `q` an import qualifier or alias | `q`'s exported non-receiver phrases |
      | `x.` with `x` not a qualifier | own receiver phrases and own `Node` fields |
      | `x.q.` | `q`'s exported receiver phrases and exported `Node` fields |

      Ranking: case-sensitive prefix, then case-insensitive prefix; then the
      own package, the imports in import order, then source order; at most 50
      items.
    - *Inserting.* The partial is replaced by the items joined with single
      spaces; for a receiver item nothing before the last `.` is touched.
      Each hole becomes a placeholder range holding the hole's name; the first
      is selected, Tab and Shift-Tab cycle, typing replaces the selection, and
      ranges move with edits like tokens. Esc, or an edit outside every range,
      clears them. Block holes are not inserted. Placeholders are drawn as a
      rounded Quiet background and are never written as text.
    - *Popup.* A borderless child window below the caret that takes no focus.
      A row shows the pattern, then `gives …; contexts` dim, and the doc line
      in a detail strip. ↑ and ↓ move, ⏎ or Tab accepts, Esc closes. It opens
      after `.` following a word or after two prefix characters; ⌃Space opens
      it anywhere.
- **Runner (M8).** Every run is `synod check --json --tokens <program>` and, when it
  reports no error, `synod run <program> --trace <file> --scene <file> --style
  short --color never`, both from the module root (decision 74). The files go
  to the app's cache folder (`~/Library/Caches/io.github.synodlang.synod/`),
  never to the workspace's `out/`. A change to a `.synod` file, `synod.mod` or
  `vendor/` (debounced 300ms), Run, a mode change or a seed change starts a new
  run and cancels the one in flight; only the newest result is applied. On
  success the scene is swapped and the viewer keeps the current fact (the same
  `seq` through the `trace` map of `04-formats.md`) when the program is
  unchanged and the `seq` exists, else the current presentation time. On
  diagnostics or a failed run the last good scene keeps playing, marked stale,
  beside the Problems list. M8 re-runs the whole program per change and does
  not use `synod run --json-events` or `--watch` (`05-cli.md`).
- **Problems and source (M8).** The Problems list shows the diagnostics of
  `synod check --json` and the `failed at` and `look failed at` lines of
  `synod run`, each with severity, code, message, position, hint, labels and
  help text. In M8 no edit is applied and clicking one opens the file read-only
  with the span highlighted; in M9 it opens the editor at the span, and a help
  edit is applied only when the user picks it (decision 81).
- **Player (M8).** A `WKWebView` hosting the bundled `player` library
  (offline), fed `scene.json` through a JS bridge; the same renderer as
  the site. The library draws by the reader rules of `04-formats.md`:
  keyframes with smootherstep, colors in OKLab, no shadows except glow,
  rounded corners on, 3Blue1Brown-style dark ground from the theme. Swift
  loads it with `SynodPlayer.mount("#stage", opts)` and then
  `player.setScene(<json string>)`; `attachWebKitBridge` posts the player's
  events (time, marker, fact, play, pause, scene) to the host.
- **Timeline (M8).** Play, pause, step to previous/next fact, scrub, speed
  (0.25× to 4×), ordered/proportional toggle, markers for `pause`
  (from `lesson.Ask`), `caption`, `broken` (the scene `markers` in
  `implementation/04-formats.md`). The speed is the player's playback rate
  and changes at once; it is not the scene's `--pace`, which the app does not
  expose in M8 (every run uses the default pace, 1). The ordered/proportional
  toggle changes the scene, so it re-runs. Markers fire when playback crosses
  them, never on a seek.
- **Inspector (M8).** The selected node's exported state at the current time;
  the trace list around the current time; clicking a fact jumps there. In M8
  the node is chosen from a list. In M9 a click on a node on the stage selects
  it, using `Player.hitTest` and the `node` shape key (decisions 83 and 84),
  and a click on a message jumps to its `net.Sent` fact (the `msg` key); the
  selected node gets an outline from `Player.nodeBounds`.
- **Export (M8: video; M9: GIF).** Video via `AVAssetWriter` (H.264/HEVC,
  1080p/4K, 30 or 60 fps, BT.709) from the `player`'s `renderFrame(at)`,
  the one render path. A separate offscreen web view holds a player on its
  own canvas; the host asks it for each frame as a lossless PNG at a fixed
  time and appends it to the video (decision 75). The speed defaults to the
  timeline's current speed. With duration `D` ms, rate `r` and `fps`, frame
  `k` shows `at_k = min(k·r·1000/fps, D)` and has presentation time `k/fps`,
  and there are `N = ceil(D·fps/(1000·r)) + 1` frames, so the first frame
  shows time 0 and the last the end. 4K uses the 1080p layout at pixel ratio
  2, letterboxed on `Paper` (`04-formats.md`, View). The scene text is
  snapshotted when the export starts; cancelling deletes the partial file.
  - *GIF (M9, decision 88).* The same schedule over a range `[a, b]` of
    presentation time, by default the whole scene `[0, D]` (the sheet also
    offers "from the playhead, for a duration"): `N = ceil((b−a)·fps/(1000·r)) + 1`
    frames, frame `k` at `at_k = min(a + k·r·1000/fps, b)`. The frame rate is
    10, 20, 25 or 50 (default 20) and every frame's delay is `100/fps`
    hundredths of a second (10, 5, 4 or 2). The width is 480, 640 (default),
    800 or 960 px, the height `round(W·canvas.h/canvas.w)`, at pixel ratio 1
    and without letterbox. At most 6,000 frames; above that the sheet asks for
    a shorter range or a lower rate; it shows an estimated size. The rate
    defaults to the timeline speed. The frames are the `renderFrame(at_k)`
    frames above. The app encodes GIF89a itself, frame by frame, so memory does
    not grow with the length: one global palette of at most 256 colors derived
    from the scene `theme`, in file order (Paper first, then the other theme
    colors, then OKLab mixes of Paper and of Ink with them),
    each pixel mapped to its nearest entry in OKLab (ties to the lower index)
    with no dithering, frames after the first reduced to the rectangle that
    changed, disposal 1, LZW with 12-bit codes, and the NETSCAPE2.0 loop count 0
    (forever). The same input gives the same bytes; cancelling deletes the
    partial file.

## Learning modes

- **Predict (M8 minimal; choices in M9).** Playback stops at every `pause`
  marker (`lesson.Ask`; the player option `stopAtPause`). In M8 the viewer
  types an answer. In M9, when the marker has `choices`
  (`lesson.Ask … choosing`, decision 82) the panel shows the question with one
  button per choice (keys 1 to 6) instead of the text field; a marker without
  `choices` keeps the field. The app does not grade the pick: playback
  continues and shows what really happened. Answers are stored per program,
  outside the workspace, in `~/Library/Application Support/Synod/answers`,
  keyed by the module root, the program and the question text (decision 77);
  with choices the entry also records the index (`choice`) and the list
  (`choices`), and `answer` is the chosen text.
- **Break it (M9).** Clicks on the stage turn into ordinary Synod
  (decisions 85 and 86); no LLM is called. The stage tells the app which node
  or message was clicked (`hitTest`), and a dragged box which nodes it holds
  (`nodeBounds`).
  - *Actions.* On a node that is up: Crash. On a crashed node: Restart. Drag
    a box around nodes: Partition, the nodes whose box center is inside against
    every other node not removed (at least one on each side); Heal while a
    partition is active. Click a message dot: Drop this message.
  - *The edit.* Each action appends one block at the end of `define main` in
    the main file (2-space indentation; a missing `use std/sim` or `use
    std/net` is added, `synod fmt` sorts it):

    ```
      after 6001ms
        sim.Crash (item 4 of sim.Nodes) # n5
    ```

    A node is `(item i of sim.Nodes)`, `i` being its 0-based index among the
    nodes not removed at that time, in creation order (from the trace), with the
    name in a trailing comment. `sim.Crash`, `sim.Restart`,
    `net.Partition {…} from {…}` and `net.Heal` use `T`, the first whole
    millisecond after the current fact's time (`0s` with no fact); the sheet
    pre-fills it and lets the user change it. Drop this message is
    `net.DropNext "<Type>" from (item a of sim.Nodes) to (item b of sim.Nodes)`,
    with `<Type>` the unqualified type of the `net.Sent` of that message and `T`
    that message's send time, written canonically (`20.431ms`), because
    `net.DropNext` acts at send time. It is offered only when no earlier message
    of that type on that link was sent at that instant; otherwise it is
    disabled with "Cannot single out this message". A top-level `after T` runs
    before any node event at `T`, so the drop is in place when the message is
    sent.
  - *Flow.* An action is offered only when the last check had no error and the
    main file is saved. The edit goes through the editor's buffer, so it is
    undoable with ⌘Z and the banner's "Undo Break it"; then the file is saved
    (formatted) and the program re-runs. The viewer keeps the presentation time.
    This is one of the places where the app writes into the workspace (see
    "Project files"), and it only happens on the user's click.
  - *Checking it.* After the run the app looks for the action's fact in the new
    trace: `sim.Crashed` or `sim.Restarted` for that node at `T`,
    `net.Partitioned` or `net.Healed` at `T`, a `net.Dropped` of that type on
    that link sent at `T`. If it is missing the app says "Break it had no
    effect", with Undo. A rule broken in the new run but not in the previous one
    (compared by its `rule` text) is shown: a red banner "Your change broke:
    <rule> at <simulated time>" with "Jump there" (to the `sim.Broken` fact), a
    Problems row, the rule's `path:line` opened in the editor with a red line
    highlight, and a pulsing timeline marker. The previous good run is kept for
    "Compare with before".
- **Compare (M9).** Two runs side by side on one timeline in simulated time
  (decision 87). Each side is a program, a seed and a mode; presets are the
  same program with the next seed, ordered against proportional, another
  program of the workspace, and "Compare with before" after a Break it. Nodes
  are matched by name (the `node` key), not by shape id, since ids are hashes
  of the draw site. The left side drives: its presentation time `at` gives the
  simulated time `T` (the smallest `T` whose map value is at least `at`), and
  the right side seeks to its own map value of `T`; the map is the
  piecewise-linear one through (0, 0), (`t`, the largest scene `trace` slot of
  the facts at `t`) for each fact time and (end, duration). A "presentation
  time" sync is also offered: the right side seeks to the smaller of the left
  time and its duration. The timeline shows both runs' markers, and the
  inspector two columns for the selected node name with the differing fields
  highlighted. Predict and Break it are off in Compare.

## Claude

See `07-claude-integration.md`. The prompt bar (M10, decision 96) runs the
user's own Claude Code CLI; there is no in-app model client, no API key and no
Keychain use, and the app never asks for, stores or passes a credential:
Claude Code keeps its own login. Generated code is just Synod source the user
owns.

- *Opt-in.* In Settings, Claude, "Enable the prompt bar (uses Claude Code)" is
  off by default. Turning it on shows a sheet saying what happens: the app runs
  your `claude` command in the workspace; your request and the files Claude
  reads go to Anthropic under your Claude Code account; the app stores no key.
  The section also has a `claude` path field (found in `~/.local/bin/claude`,
  `~/.claude/local/claude`, `/opt/homebrew/bin/claude` and
  `/usr/local/bin/claude`), an "In use" line with the output of `claude
  --version` (5 s timeout) and a Model field whose default is `claude-opus-5-5`
  (empty means Claude Code's own default). Settings are stored in
  UserDefaults; the external-session toggle below never is. The app has no
  telemetry.
- *Invocation, environment and tools.* As in `07-claude-integration.md`: the
  module root is the working folder, the bundled `synod` comes first on `PATH`,
  the bundled agent-skills SKILL.md is the system prompt, the environment is an
  allowlist, and Claude changes the workspace only through the app's editor MCP
  server and `synod mcp` (decisions 98 to 100). Claude's own Write and Edit are
  off, and the only shell command is `synod get`.
- *External sessions.* The Settings section also has "Let Claude Code in
  Terminal use the editor", off at every launch and never stored, with a "Copy
  MCP config" button and a note that the copied line is valid until Synod quits
  or Disconnect is pressed. While it is on, the window shows "Claude (Terminal)
  connected" with Disconnect, which closes the connections and rotates the
  token. The bridge is `Contents/Helpers/synod-editor-mcp`.
- *Flow.* The bar is disabled on the read-only bundled examples. Dirty buffers
  must be saved first ("Save All and Send" sheet), so Claude starts from buffers
  equal to the disk. The app snapshots the workspace's `.synod` files,
  `synod.mod` and `synod.sum` outside `vendor/`, `out/` and dot or underscore
  folders (files of at most 1 MB, 20 MB in all; above that Revert is off), and
  the module's editor buffers are read-only to the user while Claude runs.
  Claude's edits appear live in the editor, each one an undoable "Claude Edit"
  group, and stay unsaved until Claude calls `save_files` or the user saves. The
  stream becomes progress lines ("Read twopc/twopc.synod", "mcp__synod-editor__apply_edits",
  "Ran synod check .") and the final text. Stop sends SIGINT, then SIGTERM after
  3 s, then SIGKILL 2 s later. When Claude finishes, the program list refreshes,
  the newest added or changed main file's program is selected and opened, a run
  starts when its files are saved, and a banner reads "Claude changed N files"
  with "Show changes", "Save All" (while a buffer is unsaved) and "Revert".
  Revert restores the snapshot bytes and deletes added files, and runs only on
  the user's click. A follow-up uses `--resume <session_id>` from the
  `system/init` event; "New conversation" clears it. The preset "Explain this
  run" (enabled when a run exists) asks for `lesson.Note` captions in
  `define main`, without changing the behaviour.
- *Errors.* `claude` not found: setup text with the install link. An
  `is_error` result, or a non-zero exit without a result: the result text, or
  the last line of stderr. Not logged in: "Run `claude` once in Terminal to log
  in."
- *Parts.* A pure `ClaudeInvocation` (argv and environment), a tolerant
  `ClaudeStreamParser` for the stream-json events, a `ClaudeProcess` actor, a
  `ClaudeLocator` and a `WorkspaceSnapshot` (snapshot, diff, revert).

## Project files

A workspace is a folder with `synod.mod`. The app never stores Synod code
anywhere else, so everything works with Git and the CLI. Claude Code, run
from the prompt bar at the user's request, changes the workspace through the
editor's own tools (decisions 96 and 99). The app writes nothing into the
workspace on its own: run outputs and the editor's overlay files go to its cache
and Predict answers to Application Support (decisions 74, 77 and 81). It writes
a workspace file only on Save in the editor (a quick fix changes the buffer, not
the file), on a Break it action, on Revert after a prompt-bar change, or when
Claude calls `save_files` or `create_file` during a request or an external
session the user allowed.

## Distribution

The app's releases are `v0.2.x`, numbered with the language (decision 61).
Each release is `Synod-<tag>-macos.zip` with a `SHA256SUMS`, ad-hoc signed and
not notarized, and its notes name the bundled `synod`, `player` and examples.
The app bundles a pinned `synod` (the darwin-universal release archive), a
pinned `player` release and the examples at a pinned commit, listed in
`deps.env` and verified by `SHA256SUMS` and the pinned hashes (decision 76).
The bundle also holds the editor MCP bridge, `Contents/Helpers/synod-editor-mcp`
(signed like `synod`, `07-claude-integration.md`).
Because the app is not notarized, macOS blocks the first launch: the README
says to use System Settings, Privacy & Security, "Open Anyway" (on macOS 15
and later right-click, Open no longer skips the check), or to run
`xattr -dr com.apple.quarantine /Applications/Synod.app`. A Homebrew cask and
Developer ID notarization come later.
