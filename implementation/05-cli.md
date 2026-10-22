# The `synod` command

| Command | What it does |
| --- | --- |
| `synod new <dir> [--module path] [--lib]` | Create a module: `synod.mod`, `.gitignore`, a package folder with a library file and a `demo.synod` main file (`--lib`: no main file) |
| `synod fmt [paths…] [--check] [--style rich\|short] [--color auto\|always\|never]` | Rewrite files into canonical form; `--check` exits 1 if anything would change |
| `synod check [paths…] [--json] [--tokens] [--overlay file] [--style rich\|short] [--color auto\|always\|never]` | Load, resolve and check; print diagnostics |
| `synod run <folder> [--seed n] [--trace file] [--scene file] [--pace x] [--mode ordered\|proportional]` | Check and run the folder's main file: simulate and build the scene; default outputs `out/trace.jsonl` and `out/scene.json` |
| `synod run <folder> --summary` | Print a compact, human-readable summary of the run: facts grouped per step, invariant results (used by agents) |
| `synod get [module@version]` | Make `vendor/` and `synod.sum` match the build list, downloading what the store lacks; with an argument, first add or replace that `require` line |
| `synod doc [package] [--json] [--catalog] [--scope file] [--overlay file] [--style rich\|short] [--color auto\|always\|never]` | Print a package's phrases, records and `Node` fields from its source; with no package, list the visible packages; `--catalog` prints the std catalog as Markdown for `agent-skills` |
| `synod explain [code] [--json] [--catalog] [--style rich\|short] [--color auto\|always\|never]` | Explain a diagnostic code with an example and its fix; with no code, list the codes |
| `synod version` | Language, compiler and format versions (the `formats` line lists the formats the compiler writes, `synod-trace/1 synod-scene/1 synod-check/1 synod-doc/1 synod-explain/1`, so not `synod-overlay/1`, which it only reads) |

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
  exit 2, `synod check: overlay: <key>: no such file`. Only `check` and `doc`
  take it: `fmt`, `run` and the other commands exit 2 on it as on any
  unknown flag.

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
The compiler reads `vendor/` only, never the store. `synod fmt` applies the fixes that need declarations
(`spec/10`) when the package loads, and leaves them alone otherwise.

`synod new <dir> [--module path] [--lib]` details (decision 94):

- `dir` must not exist or must be an empty folder (parents are created);
  otherwise exit 2, `synod new: <dir> is not empty`. Nothing is overwritten.
- The module path is `--module` or the base name of `dir`, and must be a
  valid module path. The package name is the path's last segment and must be
  a lower-case Synod word that is not reserved; otherwise exit 2,
  `synod new: "<x>" is not a package name; use a lower-case word`.
- It writes `synod.mod` (`module <path>`, `synod 0.2`), `.gitignore`
  (`vendor/` and `out/`), `<pkg>/<pkg>.synod`, a node that counts, and,
  unless `--lib`, `<pkg>/demo.synod`, which spawns it, crashes it and stops
  the run. All pass `synod fmt --check`, `synod check` and `synod run <pkg>`.
- It prints `created <dir>: synod.mod, .gitignore, <pkg>/<pkg>.synod,
  <pkg>/demo.synod` and exits 0.

`synod get [module@version]` details (decisions 60 and 95):

- `synod get` makes `vendor/` and `synod.sum` match the build list.
  `synod get <module>@<vX.Y.Z>` first adds or replaces that `require` line in
  `synod.mod` (sorted, canonical), then does the same. A missing or malformed
  version is exit 2, `synod get: write <module>@vX.Y.Z`. It works on the
  module found in the working directory or above; without one, exit 2,
  `synod get: no synod.mod here or above`.
- A module comes from `https://` plus the first three segments of its path,
  at the tag `<version>` (`<subfolder>/<version>` for a module in a
  subfolder), by `git clone --quiet --depth 1 --no-tags --branch <tag>` with
  `GIT_TERMINAL_PROMPT=0`. No `git` on `PATH` is exit 2, `synod get:
  downloading needs git on PATH`. The module root must hold a `synod.mod`
  whose `module` line equals the requested path, else exit 2, `… declares
  module <x>`; a symbolic link in the tree is exit 2. `.git` and any
  subfolder with its own `synod.mod` are left out.
- The store `~/.synod/cache/<module>@<version>/` (on Windows under the user
  profile) is written once: an entry is built in `~/.synod/cache/.tmp-<random>/`
  and renamed into place, and an entry that already exists is used after
  verification. Entries are read-only (files 0444, folders 0555) and never
  edited.
- `synod.sum` is `04-formats.md`. A mismatch of the tree hash or of the
  commit is exit 2, `synod get: <m>@<v>: checksum mismatch: synod.sum has
  h1:…, the store has h1:…`; the store entry is never repaired or deleted (the
  message says to delete it by hand and run again).
- Minimal version selection runs over the main module's `require` lines and
  each dependency's `synod.mod` read from the store. `vendor/<module>@<version>/`
  is a copy of the verified store entry (never a link, so an edit in `vendor/`
  cannot reach the store) and is replaced when its hash differs from the sum.
  Anything else under `vendor/` is removed: `vendor/` belongs to `synod get`.
- It succeeds without network exactly when the store has every module of the
  build list. Output is one line per module, `<m>@<v> downloaded`,
  `<m>@<v> from cache` or `<m>@<v> up to date`; exit 0.

`synod doc [package]` details (decisions 90 to 92):

- An argument starting with `std/` names a std package (`std/` is the
  reserved prefix of `spec/06` §6.2; write `./std/x` for a local folder of that
  name). Otherwise an existing folder (`twopc`, `./twopc`) names that folder's
  package, and an argument that is no folder is an import path, resolved in
  the main module or `vendor/`. A bare word that is not a folder is exit 2,
  `synod doc: no package "net"`, with the hint `did you mean "std/net"?`.
- With no package it lists the visible packages, one per line:
  `<import path>  <title>` (two spaces; the title only when it is not
  empty). Order: std in byte order, then the main module's library
  packages, then the build list's packages. Without a `synod.mod` it lists
  std and the current folder.
- Short form, per phrase, no blank lines:

  ```
  net.Send [m: msg] from [a: node] to [b: node]
      action; node; Send through the network from `a`, which must be the receiver
  net.Id [m: msg]
      gives number; any; The message id of the net fact that bound `m`
  ```

  The second line is four spaces, then `action`, `gives <Gives>`,
  `node event` or `trace event`; then `; <contexts>` (std only) and
  `; <doc>` (when the doc comment is not empty). Records print as `record
  lesson.Asked: Question text, Choices list of text`, node fields as `node
  twopc.Node: State text, Participants list of twopc.Node`, and a package with
  observers or invariants adds `importing activates <n> observers, <m>
  invariants`. The rich form (terminal) has a bold `std/net — the simulated
  network` heading, a blank line between entries, the pattern in bold and the
  second line dim; color follows the rules above. Lines are not wrapped.
- Each pattern is printed by the compiler's one pattern printer, with the
  qualifier the reader writes. Only exported names are listed; `--scope`
  adds the file's own package with all its names.
- A documented package with diagnostics prints them as `synod check` does and
  exits 1. Usage and I/O problems exit 2.
- `--json` prints `synod-doc/1` (`04-formats.md`).
- `--scope <file>` needs `--json`, else exit 2. It lists what that file can
  write, spelled as that file must write it (decision 92); `--overlay` makes
  it read unsaved text. This is what the Mac app's phrase completion uses.
- `--catalog` takes no package and cannot be combined with `--json` or
  `--scope` (exit 2). It prints `# Synod standard library`, the line
  ``Generated by `synod doc --catalog` for Synod 0.2. Do not edit.``, then one
  `## std/<p> — <title>` section per std package in byte order, each with the
  table `| Phrase | Gives | Context | Meaning |` (qualified patterns; `look`
  in the context column for std/look), the `Node event:` and `Trace events:`
  lists and, for std/theme, the `| Name | Value | Use |` table. Unexported
  phrases are left out. It becomes
  `agent-skills/skills/synod/reference/std-catalog.md`.

`synod explain [code]` details (decision 93):

- Without a code, one line per code, `<code>  <summary>`, in the order of
  the `spec/11` tables (compile-time codes, then run-time codes).
- With a code, the short form is `<code>: <summary>`, a blank line, the
  prose, `Example (p/p.synod):` with each file indented 4, `synod check says:`
  (or `synod run says:`) and the diagnostic indented 4, then `Fix: <sentence>`
  and the fixed files indented 4. The rich form is the same with color and the
  diagnostic in its rich form.
- The diagnostic is produced when the command runs, by checking (or running)
  the example in memory, never read from a stored text.
- An unknown code is exit 2, `synod explain: unknown code "x"`, plus
  `  hint: did you mean "y"?` when one code is at edit distance 2 or less.
- `--json` prints `synod-explain/1` (`04-formats.md`). `--catalog` prints
  `# Synod diagnostic codes`, the generated line and the table
  `| Code | Meaning | Fix |` in the order of `spec/11`; it becomes
  `agent-skills/skills/synod/reference/errors.md`.

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
