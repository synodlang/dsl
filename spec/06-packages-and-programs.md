# 6. Packages, modules and programs

Synod follows Go.

## 6.1 Packages

- A package is a folder. All `.synod` files in it share one namespace.
- Every file starts with a comment naming its package, which must equal
  the folder name (or `main`):
  ```synod
  # package gossip
  ```
- Inside a package, its own names are bare. Outside, they are always qualified.

## 6.2 Modules

- A **module** is a Git repository (or a subfolder of one) with a
  `synod.mod` file at its root:
  ```
  module github.com/synodlang/examples
  synod 0.1
  require github.com/someone/raft v1.2.0
  ```
- `synod.sum` pins the exact commit of every dependency, so a scenario
  replays identically years later.
- Packages come **only from Git URLs**, Go style. Versions are Git tags
  (`v1.2.0`). There is no central registry.
- The standard library is the module `github.com/synodlang/std`, imported
  with the short prefix `std/` (`use std/net`). It is versioned with the
  language.
- Downloaded modules are cached in `~/.synod/cache`, so everything works
  offline after the first fetch.

## 6.3 Imports

```synod
use std/net
use github.com/synodlang/examples/gossip
```

- The qualifier is always the last path segment: `net.Send`, `gossip.Join`.
- `use … as` is legal **only** when two imports share a last segment, to
  rename one of them. Anywhere else it is an `unneeded-alias` error.
  ```synod
  use github.com/a/raft
  use github.com/b/raft as braft
  ```
- Unused imports are errors, with one exception: importing a package
  that contains observers or invariants activates them, so that import
  counts as used (`use github.com/synodlang/examples/raftlook`).
  Import cycles are errors.
- Imports come first in the file, one per line, sorted by path.

## 6.4 Exports

- Capitalization has exactly one meaning: **exported**.
- It applies to phrase heads, record types, record fields and node state fields.
- Only exported names are visible from other packages, including from look packages.

## 6.5 Programs and `main`

- A **library** is a package without `main`. A **program** is a package
  named `main` with exactly one `main` block.
- Importing a package that has `main` is an `import-main` error.
- `synod run <folder>` runs a program: it executes `main` at simulated
  time 0 in main context, then runs the event loop (`spec/07`).

```synod
# package main
use github.com/synodlang/examples/gossip
use std/sim

main
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
