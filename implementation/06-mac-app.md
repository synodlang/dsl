# Mac app (repo `mac-app`)

SwiftUI, macOS 15+, Apple silicon first. It bundles the `synod` binary
and treats it as its engine. Offline-first: everything except "Ask
Claude" works without a network.

## Main window

```
┌───────────────┬──────────────────────────────────────────────┐
│ Sidebar       │ Stage (player)                               │
│  Programs     │                                              │
│  Packages     │     n1 ●───────● n2        "Who becomes…?"   │
│  Examples     │        ╲  •   ╱                              │
│               │         ● n3                                 │
│               ├──────────────────────────────────────────────┤
│               │ Timeline: ▶ ⏸  ◀ step ▶   pace 1×   ordered ▾ │
│               ├──────────────────────────────────────────────┤
│               │ Editor (.synod) │ Inspector (state, trace)   │
└───────────────┴──────────────────────────────────────────────┘
```

## Components

- **Editor.** TextKit 2 with syntax highlighting from
  `synod check --json --tokens`, inline diagnostics, format on save,
  go-to-definition across packages, phrase completion from `synod doc`.
- **Runner.** On save (debounced 300ms) runs
  `synod run --scene … --trace …`; on success swaps the scene, keeping
  the current presentation time when possible.
- **Player.** A `WKWebView` hosting the bundled `player` library
  (offline), fed `scene.json` through a JS bridge; the same renderer as
  the site. The library interpolates keyframes with smootherstep; colors
  in OKLab; shapes have soft shadows off, rounded corners on,
  3Blue1Brown-style dark ground from the theme.
- **Timeline.** Play, pause, step to previous/next fact, scrub, pace
  (0.25× to 4×), ordered/proportional toggle, markers for `pause`
  (from `lesson.Ask`), `caption`, `broken` (the scene `markers` in
  `implementation/04-formats.md`).
- **Inspector.** The selected node's exported state at the current time;
  the trace list around the current time; clicking a fact jumps there.
- **Export.** Video via `AVAssetWriter` (H.264/HEVC, 1080p/4K, 60fps),
  rendering `player` frames offscreen at fixed timesteps
  (`renderFrame(at)`); and GIF.

## Learning modes

- **Predict.** Playback stops at every `pause` marker
  (`lesson.Ask`). The viewer types or picks an answer, then playback
  continues and shows what really happened. Answers are stored per
  program.
- **Break it.** Click a node: Crash, Restart. Click a message dot: Drop
  this message. Drag a box around nodes: Partition. Each action inserts
  a line into the program's `define main` at the current simulated time (for
  example `after 4200ms` → `sim.Crash (item 2 of nodes)` or
  `net.DropNext "Accept" from …`), formats, and re-runs. No LLM call.
  Invariants that break are highlighted where they broke.
- **Compare.** Two programs (or one program with two seeds) side by side
  with a shared timeline; shapes are matched by id.

## Claude

See `07-claude-integration.md`. The app shows a prompt bar ("animate
Raft with 5 nodes and a leader crash"), streams progress, and opens the
result in the editor. Generated code is just Synod source the user owns.

## Project files

A workspace is a folder with `synod.mod`. The app never stores Synod code
anywhere else, so everything works with Git and the CLI.
