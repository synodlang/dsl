# Publish a package

A Synod package is a folder, a module is a Git repository (or a subfolder of
one) with a `synod.mod` at its root, and there is no registry: other people
require your module by its Git URL and a tag. The rules are in
[packages, modules and programs](../../spec/06-packages-and-programs.md); this
guide is the task.

## 1. Start a module

```sh
synod new pulse --module github.com/you/pulse
cd pulse
```

`synod new` writes `synod.mod`, a `.gitignore` that holds `vendor/` and
`out/`, a package folder and a demo. The module path is the address people use,
so write the URL of the repository you will push to:

```
module github.com/you/pulse
synod 0.3
```

A module comes from `https://` plus the first three segments of its path
(`github.com/you/pulse`), fetched with `git`.

## 2. Make the package a library

Everything in the package folder that people should import goes in library
files, with the header `# package <folder>`. Name things for the outside:

- Capitalized names are exported, and only those are visible from other
  packages. `Node has` fields, phrases and records you want others to use start
  with a capital letter.
- A comment on the lines directly above a declaration, with no blank line, is
  its doc comment. `synod doc` shows it, so write it for the reader.
- Keep the `demo.synod` main file. A main file is the package's program and
  is never part of the library, so it runs but cannot be imported.

Check what a user will see:

```sh
synod doc github.com/you/pulse/pulse
```

Then run the checks every release should pass:

```sh
synod fmt --check
synod check
synod run pulse
```

Put looks and invariants in their own packages (`pulselook`, `pulsechecks`)
so users choose them with a `use`.

## 3. Tag a version

Versions are Git tags such as `v0.1.0`.

```sh
git add -A
git commit -m "First release"
git tag v0.1.0
git push origin main v0.1.0
```

For a module at the root of the repository the tag is just the version. For a
module in a subfolder, the tag is `<subfolder>/<version>`, for example
`pulse/v0.1.0`. The module's `synod.mod` must name the same path as the
requested one, or `synod get` refuses it.

## 4. Use it from another module

In the module that wants it, run `synod get` with the module and the version:

```sh
synod get github.com/you/pulse@v0.1.0
```

It needs `git` and a network the first time. It does three things:

- adds `require github.com/you/pulse v0.1.0` to `synod.mod`;
- downloads the module into the shared store `~/.synod/cache/`, once for all
  your projects, and copies it to `vendor/github.com/you/pulse@v0.1.0/`
  in your module;
- writes `synod.sum`.

Then import it like any package:

```
# package main
use github.com/you/pulse/pulse
use std/sim

define main
  set first to (sim.Spawn pulse)
  sim.StopAt 3s
```

The qualifier is the last segment of the path, so avoid names that collide with
another import; `use ... as` is allowed only for a collision.

## synod.mod, synod.sum and vendor/

| File | Holds | In git? |
| --- | --- | --- |
| `synod.mod` | `module`, `synod 0.3` and one `require` line per direct dependency | yes |
| `synod.sum` | for every module in the build list: the version, the commit of its tag and a hash of its files | yes |
| `vendor/` | one copy of each module and version, at `vendor/<module path>@<version>/` | no (in `.gitignore`) |

- Commit `synod.sum`. It makes a fresh `synod get` reproducible: the first
  download of a module records its line, and afterwards a tag that moved to
  another commit, or files that differ, fail with `checksum mismatch`.
- `vendor/` is the only place the compiler reads. After a clone, run
  `synod get` with no argument to fill it. Once fetched, everything works offline.
- The versions used are chosen by minimal version selection over every
  `require` line of your module and its dependencies, so a program has one
  version of each module.
- Never move a published tag. Publish `v0.1.1` instead.

The standard library is not a dependency: `std/net` is always available and is
versioned with the language. [Chapter 14](../14-packages.md) builds a program on
top of a required package.

## Next

- [Use Synod with Claude or another agent](agents.md): an agent can find your
  package with `synod doc` and reuse it.
