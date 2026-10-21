# 6. Packages, modules and programs

Synod follows Go.

## 6.1 Packages

- A package is a folder. All `.synod` files in it share one namespace,
  except a main file (§6.5), which is a package of its own.
- Every file starts with a comment naming its package: the folder name,
  or `main` for the folder's one main file (§6.5):
  ```synod
  # package gossip
  ```
- Inside a package, its own names are bare. Outside, they are always qualified.

## 6.2 Modules

- A **module** is a Git repository (or a subfolder of one) with a
  `synod.mod` file at its root:
  ```
  module github.com/synodlang/examples
  synod 0.2
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
- An import is used when the file refers to the package, or when the
  package contains observers or invariants: importing it activates them
  (`use github.com/synodlang/examples/raftlook`). Unused imports and
  import cycles are errors.
- An import path must name a library package: a folder that holds a
  library file. A missing folder, or a folder holding only a main file,
  is an `unknown-import` error.
- Imports come first in the file, one per line, sorted by path.

## 6.4 Exports

- Capitalization has exactly one meaning: **exported**.
- It applies to phrase heads, record types, record fields, node fields,
  and the node type `Node`, which is always exported (`gossip.Node`).
- Only exported names are visible from other packages, including from look packages.

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
