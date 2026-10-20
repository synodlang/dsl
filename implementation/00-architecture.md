# Architecture

## Repositories

Everything belonging to the language lives in the
[synodlang](https://github.com/synodlang) organization.

| Repository | Language | Contents |
| --- | --- | --- |
| `dsl` | Markdown | This specification, the std catalog, these guides, decisions |
| `compiler` | Go | The `synod` command: loader, parser, formatter, resolver, checker, simulator, scene builder, doc generator |
| `std` | Synod (+ native ids) | The standard library packages; natives are declared here and implemented in `compiler` |
| `examples` | Synod | Sample packages and programs (start from `dsl/examples/`) |
| `mac-app` | Swift / SwiftUI | Editor, player, Predict / Break it / Compare, Claude CLI integration, video export |
| `agent-skills` | Markdown | Skills that teach coding agents to read and write Synod |
| `homepage` | Static site | synodlang website: intro, docs rendered from `dsl`, recordings |
| `.github` | YAML / Markdown | Organization profile, shared CI workflows, contribution guide |

Rules:

- `dsl` is versioned (`v0.1.0`, …). `compiler`, `std` and `agent-skills`
  each declare which `dsl` version they implement.
- The **trace format** and **scene format** (`04-formats.md`) are
  contracts between `compiler` and `mac-app`, versioned in `dsl`.
- `agent-skills` must be regenerated whenever `dsl` or `std` changes; CI in
  `agent-skills` fails if it is stale.
- Official packages are imported as `github.com/synodlang/<repo>/<package>`.

## Data flow

```
             ┌──────────── compiler repo: the `synod` binary (Go) ─────────────┐
 .synod ───► │ load ─► parse ─► format check ─► resolve ─► check ─► IR         │
 synod.mod   │                                                  │              │
             │                     simulate (sim clock) ◄───────┘              │
             │                        │                                        │
             │                     trace.jsonl ─► scene builder (look ctx) ─►  │ ─► scene.json
             └─────────────────────────────────────────────────────────────────┘
                                                                     │
 mac-app (Swift): editor ─► runs `synod` ─► reads scene.json ─► player / video export
                      ▲
                      └─ Claude CLI writes .synod source on request
```

- The Mac app never interprets Synod. It shells out to the bundled
  `synod` binary and renders the scene.
- Everything is deterministic: same sources, seed and version produce
  byte-identical trace and scene files.

## Why Go for the compiler

A single static binary for macOS (arm64, amd64) and Linux; fast enough to
re-run a whole simulation on every edit (target: a 6-node, 30-second
Cassandra run in under 200ms); and it is the maintainer's language. The
app bundles a universal `synod` binary and calls it as a subprocess.

## Performance targets

| Operation | Target |
| --- | --- |
| `synod check` on all examples | < 100ms |
| `synod run` 6 nodes, 30s simulated, ~5,000 messages | < 200ms including scene |
| Player | 120fps on Apple silicon with 50 nodes and 200 messages in flight |
