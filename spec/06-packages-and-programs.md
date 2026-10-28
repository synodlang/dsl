# 6. Packages, modules and programs

Synod follows Go.

## 6.1 Packages

- A package is a folder. All `.synod` files in it share one namespace,
  except a main file (§6.5), which is a package of its own.
- Every file starts with a comment naming its package: the folder name,
  or `main` for the folder's one main file (§6.5). The header is exactly
  `# package <word>` on the first non-blank line:
  ```synod
  # package gossip
  ```
  A **theme** file (§6.7) has the header `# theme <word>` instead, the word
  being the folder name. All library files of a folder have one header
  kind.
- Package discovery skips folders named `vendor` or `testdata`, and
  folders starting with `.` or `_`.
- Inside a package, its own names are bare. Outside, they are always qualified.

## 6.2 Modules

- A **module** is a Git repository (or a subfolder of one) with a
  `synod.mod` file at its root:
  ```
  module github.com/synodlang/examples
  synod 0.3
  require github.com/someone/raft v1.2.0
  ```
- `synod.sum` pins the exact commit of every dependency, so a scenario
  replays identically years later.
- Packages come **only from Git URLs**, Go style. Versions are Git tags
  (`v1.2.0`). There is no central registry.
- The standard library is the module `github.com/synodlang/std`, imported
  with the short prefix `std/` (`use std/net`). It is versioned with the
  language.
- Downloaded modules live in a `vendor/` folder at the root of the module
  that needs them, one folder per module and version,
  `vendor/<module path>@<version>/` (e.g.
  `vendor/github.com/someone/raft@v1.2.0/`). This is the only place the
  compiler reads: there is no override. `synod get` fills it from a shared
  download store, `~/.synod/cache/<module>@<version>/`, which holds each
  downloaded module once for all projects, with the commit of its tag
  beside it, verified against `synod.sum` and never edited; `synod get` copies from it and downloads only what the
  store lacks. `vendor/` is ignored by git
  (`synod new` writes a `.gitignore` containing `vendor/`); `synod.sum`
  keeps builds reproducible after a fresh `synod get`, and everything
  works offline once fetched (`synod.sum` is specified in
  `implementation/04-formats.md`). Only the main module's `vendor/` is read; a
  dependency's own `vendor/` is never used. The versions used are chosen
  by minimal version selection over every `require` line of the main
  module and its dependencies, so one version per module. A folder with no `synod.mod` above it
  is a module with no path: only `std/` imports resolve in it.
- A module comes from the repository at `https://` plus the first three
  segments of its path (`github.com/someone/raft`), fetched with `git`. The
  tag is the version (`v1.2.0`) for a module at the repository root and
  `<subfolder>/<version>` for a module in a subfolder (decision 95).
- `github.com/synodlang/std/…` is never imported directly; `std/<package>`
  is the only spelling, and `@version` on a `std/` path is an error.

## 6.3 Imports

```synod
use github.com/synodlang/examples/gossip
use std/net
```

- The qualifier is always the last path segment: `net.Send`, `gossip.Join`.
- `use … as` is legal **only** on a collision of last segments: when two
  imports of the file share one, or when an import's last segment is the
  importing package's own name (a package `raft` importing another
  `raft`), to rename one of them. Anywhere else it is an `unneeded-alias` error.
  ```synod
  use github.com/a/raft
  use github.com/b/raft as braft
  ```
  An `@version` on a `use` path must equal the version the build selects.
- An import is used when the file refers to the package, when the
  package contains observers or invariants (importing it activates them,
  `use github.com/synodlang/examples/raftlook`), or when the package is a
  theme (importing it selects it, §6.7). Unused imports and
  import cycles are errors.
- An import path must name a library package: a folder that holds a
  library file. A missing folder, or a folder holding only a main file,
  is an `unknown-import` error.
- Imports come first in the file, directly after the header, one per line, sorted by the byte order of the full path text (including any `@version`). A `use` after another declaration is moved up by `synod fmt`.

## 6.4 Exports

- Capitalization has exactly one meaning: **exported**.
- It applies to phrase heads, record types, record fields, node fields,
  and the node type `Node`, which is always exported (`gossip.Node`).
- Only exported names are visible from other packages, including from look packages.
- A declaration's **doc comment** is the comment lines directly above its
  line, with no blank line between; `synod doc` shows it (decision 90).

## 6.5 Programs and `define main`

- A **main file** is a file whose header is `# package main`. It is a
  package of its own, the folder's **program**. The folder's other files
  form its library package, named after the folder. A folder holds at
  most one main file.
- A main file contains exactly one `define main`, and `define main`
  appears only in main files.
- The main file uses its folder's library like any other importer,
  qualified (`use github.com/synodlang/examples/gossip`).
- Importing a folder imports its library package; the main file is never
  part of it. A folder holding only a main file (such as
  `examples/webapp`) is a pure program and cannot be imported.
- A main file declares no receivers, no `define [n: Node] start`, no
  node-event `when` and no `Node has`, because a main package never runs
  on nodes (`bad-receiver`).
- `synod run <folder>` runs the folder's main file: it executes
  `define main` at simulated time 0 in main context, then runs the event
  loop (`spec/07`).

```synod
# package main
use github.com/synodlang/examples/gossip
use std/sim

define main
  set nodes to (sim.Cluster of 6 running gossip)
  each i, n in nodes
    if (i more than 0)
      gossip.Join n to (item 0 of nodes)
  after 10s
    sim.Crash (item 2 of nodes)
  sim.StopAt 30s
```

## 6.6 Which looks and checks are active

- Every `when` on a trace event and every `always`, in every package of
  the program's import graph, is active.
- If no package in the graph uses `std/look`, the scene builder uses
  `std/look/basic` automatically, so every program animates.

## 6.7 Themes

A **theme** is a library package that holds the skin of a program: its
colors, type and a few look settings (`spec/09` §9.5).

- **Header.** A theme file starts with `# theme <word>`, the word being the
  folder name. All library files of a folder share one header kind, and
  mixing kinds is `bad-header`. A folder may also hold a main file, a demo
  of its theme (§6.5).
- **Contents.** A theme file holds only definitions `define <Name>` whose
  block is exactly one `result` of a literal; doc comments are allowed.
  It has no `use`, `has`, `when`, `always`, receiver or `native`. The
  theme defines each name of the theme table (`spec/09` §9.5) exactly once,
  and each literal has the name's type and lies in its domain. Anything else
  is `bad-theme`.
- **No exports.** A theme exports no phrase: `terminal.Paper` for a theme
  `terminal` is `unknown-phrase`. Theme values are reached only as
  `theme.<Name>`, the phrases of `std/theme`, in look context.
- **Selection.** The theme of a program is the theme its main file imports,
  else `std/theme/default`. Only a main file imports a theme, and at most
  one: a library file importing a theme is `theme-outside-main`, a second
  theme in a main file is `extra-theme`. Importing a theme selects it, so
  the import is used (§6.3). `std/theme/default` is a theme like any other:
  a main file that imports it is `non-canonical`, and `synod fmt` removes
  the line, so the default has one spelling.
- **The interface.** `std/theme` is an ordinary package whose phrases are
  natives, one per name of the theme table, returning the value of the
  program's theme (`# package theme`, `define Paper native "theme.Paper"`).
  They are look-context only, so no node, main or check code reads a theme
  and the trace of a run is identical under every theme.
- **The default.** `std/theme/default` is a theme that holds the default
  value of every name; copy it to write a new theme.

```
# theme sunrise
# scene ground
define Paper
  result #FFF4E0

# scene ink
define Ink
  result #2B2B2B
```

```
# package main
use github.com/synodlang/examples/raft
use github.com/synodlang/looks/lumen
use github.com/synodlang/looks/terminal
```

The theme above is cut to two names for the page; a theme file holds all of
them. The second block selects the theme `terminal` for the program.
