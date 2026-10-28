# Install the CLI

The `synod` command formats, checks and runs Synod programs, and it is all the
tutorial needs. It is one binary with no other dependencies.

## Pick a way

The [install page](https://synodlang.github.io/install/) shows the commands for
your system and the direct download links of the latest release.

| System | Command |
| --- | --- |
| macOS, Linux (Homebrew) | `brew install synodlang/tap/synod` |
| macOS, Linux (script) | `curl -fsSL https://synodlang.github.io/install.sh \| sh` |
| Windows (Scoop) | `scoop bucket add synodlang https://github.com/synodlang/scoop-bucket` then `scoop install synod` |
| Any system with Go 1.23 or later | `go install github.com/synodlang/compiler/cmd/synod@latest` |

On macOS and Linux:

```sh
brew install synodlang/tap/synod
```

On Windows:

```sh
scoop bucket add synodlang https://github.com/synodlang/scoop-bucket
scoop install synod
```

## The install script

```sh
curl -fsSL https://synodlang.github.io/install.sh | sh
```

The script downloads the archive for your system from the GitHub release and
checks it against the release's `SHA256SUMS` before it installs anything. It
puts `synod` in `~/.local/bin`; set `SYNOD_INSTALL` to choose another folder and
`SYNOD_VERSION` to pin a version. If the folder is not on your `PATH`, the
script prints the `export PATH=...` line to add. It does not run on Windows;
use Scoop there.

## Direct download

Download the archive for your system and `SHA256SUMS` from the
[releases](https://github.com/synodlang/compiler/releases) page, check the sum,
unpack it and put `synod` on your `PATH`. The binaries are not code-signed yet,
so macOS and Windows warn about a file downloaded with a browser. Package
managers and `install.sh` avoid the warning. On macOS, a binary downloaded in a
browser needs `xattr -d com.apple.quarantine ./synod`; on Windows SmartScreen,
choose "More info" and then "Run anyway".

## Check the install

```sh
synod version
```

It prints three lines: the language version, the compiler version with its
commit, and the file formats it reads and writes. The language line should say
`language 0.2`.

## Try it

`synod new` makes a small module with a program that runs at once.

```sh
synod new hello --module example.com/hello
cd hello
synod run hello
```

`synod run` writes the trace to `out/trace.jsonl` and the scene to
`out/scene.json`, and prints one status line for each. Add `--summary` to read
the run as text.

```sh
synod run hello --summary
```

The same program and seed give the same bytes on every run and every platform.
Add `--seed 7` to change the seed.

## Next

- [The tutorial](../README.md) starts from a single sentence.
- [Use the Mac app](mac-app.md) to watch a run and to predict and break it.
- [Use Synod with Claude or another agent](agents.md) lets an assistant write
  Synod for you.
- `synod get` needs `git` on your `PATH`, but only when a module requires
  another module ([Publish a package](publish-a-package.md)).
