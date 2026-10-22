# Mac app (repo `mac-app`)

SwiftUI, macOS 15+, Apple silicon first (the bundled `synod` is the
universal binary). It bundles the `synod` binary and treats it as its
engine. Offline-first: everything except "Ask Claude" works without a
network.

**Milestones.** M8 ships the workspace, runner, player, timeline,
inspector, problems and a read-only source view, video export and a
minimal Predict. M9 adds the editor, GIF export, Predict choices, Break it
and Compare. Each component below says which milestone it belongs to.

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
editor is M9.

## Components

- **Editor (M9; M8 has a read-only source view).** TextKit 2 with syntax
  highlighting from `synod check --json --tokens`, inline diagnostics (underline from `line:col`
  to `end`, secondary `labels`, quick fixes from `help[].edits`), format on save,
  go-to-definition across packages, phrase completion from `synod doc`.
- **Runner (M8).** Every run is `synod check --json <program>` and, when it
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
  help text (no edit is applied before M9). Clicking one opens the file
  read-only with the span highlighted.
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
  the node is chosen from a list; choosing it by clicking the stage needs the
  hit test of gap G15 (M9).
- **Export (M8: video; GIF is M9).** Video via `AVAssetWriter` (H.264/HEVC,
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

## Learning modes

- **Predict (M8 minimal; choices in M9).** Playback stops at every `pause`
  marker (`lesson.Ask`; the player option `stopAtPause`). In M8 the viewer
  types an answer; M9 adds picking from choices. Then playback continues and
  shows what really happened. Answers are stored per program, outside the
  workspace, in `~/Library/Application Support/Synod/answers`, keyed by the
  module root, the program and the question text (decision 77).
- **Break it (M9).** Click a node: Crash, Restart. Click a message dot: Drop
  this message. Drag a box around nodes: Partition. Each action inserts
  a line into the program's `define main` at the current simulated time (for
  example `after 4200ms` → `sim.Crash (item 2 of nodes)` or
  `net.DropNext "Accept" from …`), formats, and re-runs. No LLM call.
  Invariants that break are highlighted where they broke.
- **Compare (M9).** Two programs (or one program with two seeds) side by side
  with a shared timeline; shapes are matched by id.

## Claude

See `07-claude-integration.md`. The app shows a prompt bar ("animate
Raft with 5 nodes and a leader crash"), streams progress, and opens the
result in the editor. Generated code is just Synod source the user owns.

## Project files

A workspace is a folder with `synod.mod`. The app never stores Synod code
anywhere else, so everything works with Git and the CLI. It writes nothing
into the workspace on its own: run outputs go to its cache and Predict
answers to Application Support (decisions 74 and 77).

## Distribution

The app's releases are `v0.2.x`, numbered with the language (decision 61).
Each release is `Synod-<tag>-macos.zip` with a `SHA256SUMS`, ad-hoc signed and
not notarized, and its notes name the bundled `synod`, `player` and examples.
The app bundles a pinned `synod` (the darwin-universal release archive), a
pinned `player` release and the examples at a pinned commit, listed in
`deps.env` and verified by `SHA256SUMS` and the pinned hashes (decision 76).
Because the app is not notarized, macOS blocks the first launch: the README
says to use System Settings, Privacy & Security, "Open Anyway" (on macOS 15
and later right-click, Open no longer skips the check), or to run
`xattr -dr com.apple.quarantine /Applications/Synod.app`. A Homebrew cask and
Developer ID notarization come later.
