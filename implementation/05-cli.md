# The `synod` command

| Command | What it does |
| --- | --- |
| `synod new <dir> [--lib]` | Create a module: `synod.mod`, a package folder with a library file and a `demo.synod` main file (`--lib`: no main file), `.gitignore` |
| `synod fmt [paths…] [--check] [--style rich\|short] [--color auto\|always\|never]` | Rewrite files into canonical form; `--check` exits 1 if anything would change |
| `synod check [paths…] [--json] [--tokens] [--overlay file] [--style rich\|short] [--color auto\|always\|never]` | Load, resolve and check; print diagnostics |
| `synod run <folder> [--seed n] [--trace file] [--scene file] [--pace x] [--mode ordered\|proportional]` | Check and run the folder's main file: simulate and build the scene; default outputs `out/trace.jsonl` and `out/scene.json` |
| `synod run <folder> --summary` | Print a compact, human-readable summary of the run: facts grouped per step, invariant results (used by agents) |
| `synod get [module@version]` | Fill `vendor/` from the shared download store (downloading what is missing) and update `synod.mod`/`synod.sum` |
| `synod doc [package] [--catalog]` | Print a package's exported phrases; `--catalog` prints the std catalog as Markdown for `agent-skills` |
| `synod explain <code>` | Explain a diagnostic code with an example |
| `synod version` | Language, compiler and format versions (the `formats` line lists the formats the compiler writes, so not `synod-overlay/1`, which it only reads) |

Folders: `synod fmt` and `synod check` walk a folder argument for `.synod`
files and skip folders whose names start with `.` or `_` and folders named
`testdata` or `vendor`; a file or folder named on the command line is always
processed.

Diagnostic output (`fmt`, `check`, `run`):

- `--style rich|short` and `--color auto|always|never`, as `--style rich` or
  `--style=rich`; another value exits 2. The short form is the one line of
  `spec/11` §11.1 (the same bytes as before M4); the rich form adds the
  source line, markers, labels, notes and help under the same first line.
- By default the style is rich when the destination is a terminal, short
  otherwise (a pipe or a file). The destination is stdout for `check` and for
  `run`, and stderr for `fmt` diagnostics and for the status of `run --trace -`
  or `--scene -`,
  so detection is on that stream. A terminal is a character device; a pipe from
  MSYS or mintty counts as a pipe.
- `--color auto` (the default) colors rich output when the destination is a
  terminal, `NO_COLOR` is unset or empty and `TERM` is not `dumb`. `always`
  and `never` beat the environment, `NO_COLOR` included. `--color` only
  matters for rich output. On Windows color switches the console to ANSI
  processing and is off if that fails.
- `--json` ignores both flags.

`synod check` details:

- `--tokens` needs `--json`, else exit 2 (the tokens are part of
  `check.json`, `04-formats.md`). A file that fails to lex or parse still gets lexical
  tokens, and so does every other file of its package (decision 79).
- `--overlay file` checks unsaved text without writing it (decision 80). The
  file is `overlay.json` (`04-formats.md`): it replaces the content of the
  `.synod` files and `synod.mod` it names, and diagnostics and tokens keep the
  replaced file's path and positions. A missing or unreadable overlay, a bad
  `format` or a key that is not an existing `.synod` file or `synod.mod` is
  exit 2, `synod check: overlay: <key>: no such file`. Only `check` takes it:
  `fmt`, `run` and the other commands exit 2 on it as on any unknown flag.

`synod run <folder>` details:

- Flags may come before or after the folder; `--seed 7` and `--seed=7`
  (decimal, 0 to 2^53; it wins over `sim.Seed`, `spec/07` §7.7).
  `--trace` defaults to `out/trace.jsonl` and `--scene` to `out/scene.json`,
  relative to the working directory (parents are created). `synod run`
  always writes both, also after a failed run. `--trace -` writes the trace
  to stdout and the status line to stderr; `--scene -` does the same for the
  scene. Both `-` is exit 2: `--trace and --scene cannot both be "-"`. An
  empty value is exit 2.
- `--pace x`: a decimal with at most 3 fraction digits from 0.25 to 4
  (stored as thousandths, `spec/09` §9.6); anything else is exit 2,
  `--pace needs a number from 0.25 to 4, not "x"`. `--mode ordered|proportional`
  (default `ordered`); another value is exit 2.
- Diagnostics print as in `synod check` (same styles); any one stops the run (exit 1, no
  trace). A folder with no main file is `missing-main`, exit 1.
- Stdout carries no wall-clock time: `ok seed 7: stop at 2s, 3 nodes, 212
  events -> out/trace.jsonl`, plus one line per broken `always`; a failed
  run prints `failed at 2s: <pos>: error[<code>]: <message>` and, in rich
  style, the source line of the failure under it (the trace and the scene
  are still written; exit 1). A broken `always` alone is exit 0.
  The `<pos>` of an error raised inside std is the user's call that entered
  std, and the rich form adds the std position as a note (decision 89).
- After those lines the scene status: `scene 18.45s, 42 shapes ->
  out/scene.json` (the duration as a time, `spec/04`; `-> stdout` for
  `--scene -`).
- A look fault (`spec/11` §11.3) replaces the scene line with `look failed at
  <simulated time of the fact>: <pos>: error[<code>]: <message>`, followed in
  rich style by the source line (as for `failed at`); the trace and the
  partial scene are still written, with a `failed` marker; exit 1.
- `--summary` prints the compact summary instead of the status lines and
  still writes the trace and the scene silently; a look fault still prints
  its `look failed at` line after the summary and exits 1.
- Until their milestones, `--watch` and `--json-events` exit 2 with
  `synod run: --watch is not implemented yet`.

Modules: `synod check` and `synod run` read remote modules from the main
module's `vendor/` only, at `vendor/<module path>@<version>/`; `synod new`
writes a `.gitignore` containing `vendor/`;
a missing one is `unknown-import` with the hint `run "synod get <m>@<v>"`.
`synod get` fills `vendor/` from the shared download store
`~/.synod/cache/<module>@<version>/` (copying, or hard-linking where
possible) and downloads only the modules the store lacks; downloads are
verified against `synod.sum`, and store entries are never edited. The
compiler reads `vendor/` only, never the store. `synod get`, network
access and `synod.sum` verification are not implemented in M2. `synod fmt` applies the fixes that need declarations
(`spec/10`) when the package loads, and leaves them alone otherwise.

Exit codes: 0 success, 1 diagnostics or a `sim.Failed` run, 2 usage or I/O error.

`synod run` with `--watch` re-runs on every save and prints one line per
run (`ok 6 nodes 4,812 events 31ms`); the Mac app may later use
`synod run --json-events` to stream progress. The app does not: it runs
`synod check --json --tokens` and `synod run` again for every change
(decision 74), so `--watch` and `--json-events` stay not implemented.

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
- `synod-<tag>.wasm` and `synod-<tag>-wasm_exec.js` (the WebAssembly build
  and its loader, `01-compiler.md`),
- `SHA256SUMS` (lists every archive and both wasm files),

and updates the `homebrew-tap` and `scoop-bucket` repositories.

| Channel | Command |
| --- | --- |
| Homebrew | `brew install synodlang/tap/synod` |
| Scoop | `scoop bucket add synodlang https://github.com/synodlang/scoop-bucket` then `scoop install synod` |
| Script (macOS, Linux) | `curl -fsSL https://synodlang.github.io/install.sh \| sh` (`SYNOD_VERSION` pins, `SYNOD_INSTALL` sets the directory; verifies `SHA256SUMS`; no sudo) |
| Go | `go install github.com/synodlang/compiler/cmd/synod@latest` |
| Download | GitHub Releases of `synodlang/compiler` |

The site's `/install/` page lists all of these (`09-site-and-org.md`).
