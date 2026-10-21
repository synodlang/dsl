# The `synod` command

| Command | What it does |
| --- | --- |
| `synod new <dir> [--lib]` | Create a module: `synod.mod`, a package folder with a library file and a `demo.synod` main file (`--lib`: no main file), `.gitignore` |
| `synod fmt [paths…]` | Rewrite files into canonical form; `--check` exits 1 if anything would change |
| `synod check [paths…] [--json] [--tokens]` | Load, resolve and check; print diagnostics |
| `synod run <folder> [--seed n] [--trace file] [--scene file] [--pace x] [--mode ordered\|proportional]` | Check and run the folder's main file: simulate and build the scene; default outputs `out/trace.jsonl` and `out/scene.json` |
| `synod run <folder> --summary` | Print a compact, human-readable summary of the run: facts grouped per step, invariant results (used by agents) |
| `synod get [module@version]` | Fetch dependencies into the cache and update `synod.mod`/`synod.sum` |
| `synod doc [package] [--catalog]` | Print a package's exported phrases; `--catalog` prints the std catalog as Markdown for `agent-skills` |
| `synod explain <code>` | Explain a diagnostic code with an example |
| `synod version` | Language, compiler and format versions |

Modules: `synod check` and `synod run` read remote modules from the cache
only, `$SYNOD_CACHE` or `~/.synod/cache`, at `<cache>/<module>@<version>/`;
a missing one is `unknown-import` with the hint `run "synod get <m>@<v>"`.
`synod get`, network access and `synod.sum` verification are not
implemented in M2. `synod fmt` applies the fixes that need declarations
(`spec/10`) when the package loads, and leaves them alone otherwise.

Exit codes: 0 success, 1 diagnostics or a `sim.Failed` run, 2 usage or I/O error.

`synod run` with `--watch` re-runs on every save and prints one line per
run (`ok 6 nodes 4,812 events 31ms`); the Mac app uses
`synod run --json-events` to stream progress.

## Distribution

Targets: macOS (amd64, arm64, and a universal archive), Linux (amd64,
arm64), Windows (amd64, arm64). Binaries are **unsigned for now**: no
Apple Developer ID or Windows code-signing certificate yet. Package
managers and `install.sh` avoid the quarantine and SmartScreen warnings;
browser downloads need `xattr -d com.apple.quarantine ./synod` (macOS) or
"More info → Run anyway" (Windows).

A release is built on every `v*` tag of `compiler`. The version comes
from the tag; the commit comes from Go build info (`synod version`). It
publishes to GitHub Releases:

- `synod-<tag>-<os>-<arch>.tar.gz` (Windows: `.zip` with `synod.exe`),
- `synod-<tag>-darwin-universal.tar.gz`,
- `SHA256SUMS`,

and updates the `homebrew-tap` and `scoop-bucket` repositories.

| Channel | Command |
| --- | --- |
| Homebrew | `brew install synodlang/tap/synod` |
| Scoop | `scoop bucket add synodlang https://github.com/synodlang/scoop-bucket` then `scoop install synod` |
| Script (macOS, Linux) | `curl -fsSL https://synodlang.github.io/install.sh \| sh` (`SYNOD_VERSION` pins, `SYNOD_INSTALL` sets the directory; verifies `SHA256SUMS`; no sudo) |
| Go | `go install github.com/synodlang/compiler/cmd/synod@latest` |
| Download | GitHub Releases of `synodlang/compiler` |

The site's `/install/` page lists all of these (`09-site-and-org.md`).
