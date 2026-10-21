# Architecture

## Repositories

Everything belonging to the language lives in the
[synodlang](https://github.com/synodlang) organization.

| Repository | Language | Contents |
| --- | --- | --- |
| `dsl` | Markdown | This specification, the std catalog, these guides, decisions, the tutorial |
| `compiler` | Go | The `synod` command: loader, parser, formatter, resolver, checker, simulator, scene builder, doc generator; also built as `synod.wasm` |
| `std` | Synod (+ native ids) | The standard library packages; natives are declared here and implemented in `compiler` |
| `examples` | Synod | Sample packages and programs; the acceptance tests of the roadmap |
| `player` | TypeScript | Canvas library that plays `scene.json`; used by `mac-app` (in a WKWebView) and the site |
| `mac-app` | Swift / SwiftUI | Editor, player host, Predict / Break it / Compare, Claude CLI integration, video export |
| `agent-skills` | Markdown | Skills that teach coding agents to read and write Synod |
| `synodlang.github.io` | Static site | Organization site at https://synodlang.github.io: intro, documentation built from `dsl` (`11-documentation.md`), interactive examples (`synod.wasm` + `player`) |
| `homebrew-tap` | Ruby | Homebrew formula for `synod`; updated by the `compiler` release |
| `scoop-bucket` | JSON | Scoop manifest for `synod`; updated by the `compiler` release |
| `.github` | YAML / Markdown | Organization profile, shared CI workflows, contribution guide |

Rules:

- `dsl` is versioned (`v0.2.1`, …). `compiler`, `std` and `agent-skills`
  each declare which `dsl` version they implement.
- The **trace format** and **scene format** (`04-formats.md`) are
  contracts between `compiler` and `player`, versioned in `dsl`.
- `agent-skills` must be regenerated whenever `dsl` or `std` changes; CI in
  `agent-skills` fails if it is stale.
- Official packages are imported as `github.com/synodlang/<repo>/<package>`.

## Data flow

```
        ┌── compiler repo: native `synod` binary and `synod.wasm` (Go) ───┐
 .synod │ load ─► parse ─► format check ─► resolve ─► check ─► IR         │
 synod.mod                                                   │            │
 ─────► │                     simulate (sim clock) ◄─────────┘            │
        │                        │                                        │
        │                     trace.jsonl ─► scene builder (look ctx) ─►  │ ─► scene.json
        └─────────────────────────────────────────────────────────────────┘
                                                                     │
                                                       player (TypeScript canvas library)
                                                          ▲                      ▲
 mac-app (Swift): editor ─► runs `synod` ─► scene.json ───┘                      │
     │   WKWebView host; video export renders player frames                      │
     └─ Claude CLI writes .synod source on request                               │
 site: editable example ─► runs `synod.wasm` in the page ─► scene.json ──────────┘
```

- `player` has two consumers. (1) The Mac app hosts it in a WKWebView:
  the editor runs the native `synod`, and video export renders player
  frames. (2) The site runs `synod.wasm` in the page, so examples are
  editable and re-runnable.
- Neither the app nor the site interprets Synod: the app calls the
  bundled `synod`, the site calls `synod.wasm`, and both draw with `player`.
- Everything is deterministic: same sources, seed and version produce
  byte-identical trace and scene files.

## Why Go for the compiler

A single static binary for macOS (arm64, amd64) and Linux; fast enough to
re-run a whole simulation on every edit (target: a 6-node, 30-second
Cassandra run in under 200ms); it is the maintainer's language, and it builds to WebAssembly with the standard toolchain. The
app bundles a universal `synod` binary and calls it as a subprocess.

## Performance targets

| Operation | Target |
| --- | --- |
| `synod check` on all examples | < 100ms |
| `synod run` 6 nodes, 30s simulated, ~5,000 messages | < 200ms including scene |
| Player | 120fps in WKWebView/Safari on Apple silicon with 50 nodes and 200 messages in flight |
| `synod.wasm` check + run + scene of any example in the browser | < 500ms |
