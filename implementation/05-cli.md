# The `synod` command

| Command | What it does |
| --- | --- |
| `synod new <dir> [--lib]` | Create a module: `synod.mod`, a `main` (or library) package, `.gitignore` |
| `synod fmt [paths…]` | Rewrite files into canonical form; `--check` exits 1 if anything would change |
| `synod check [paths…] [--json] [--tokens]` | Load, resolve and check; print diagnostics |
| `synod run <program> [--seed n] [--trace file] [--scene file] [--pace x] [--mode ordered\|proportional]` | Check, simulate, build the scene; default outputs `out/trace.jsonl` and `out/scene.json` |
| `synod run <program> --summary` | Print a compact, human-readable summary of the run: facts grouped per step, invariant results (used by agents) |
| `synod get [module@version]` | Fetch dependencies into the cache and update `synod.mod`/`synod.sum` |
| `synod doc [package] [--catalog]` | Print a package's exported phrases; `--catalog` prints the std catalog as Markdown for `agent-skills` |
| `synod explain <code>` | Explain a diagnostic code with an example |
| `synod version` | Language, compiler and format versions |

Exit codes: 0 success, 1 diagnostics or a `sim.Failed` run, 2 usage or I/O error.

`synod run` with `--watch` re-runs on every save and prints one line per
run (`ok 6 nodes 4,812 events 31ms`); the Mac app uses
`synod run --json-events` to stream progress.

Distribution: GitHub releases with universal macOS and Linux binaries,
plus a Homebrew tap `synodlang/tap`.
