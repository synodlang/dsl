# Documentation (site `synodlang.github.io`)

The site carries all Synod documentation. It is built from `dsl` and
`std` at build time, so it always matches a tagged specification.

## Sections

| Section | Source | What it is |
| --- | --- | --- |
| **Tutorial** | `dsl/tutorial/` | Learn Synod from nothing, bottom up (below) |
| **Reference** | `dsl/spec/` | The specification, one page per chapter, normative |
| **Library** | `dsl/std/README.md` and `synod doc --json` of the pinned compiler | One page per std package: every phrase with its pattern, result, contexts and a runnable example from `dsl/std/examples/` (decision 105) |
| **Examples** | `examples` repo | One page per example: editable source, Run, and the story to watch |
| **Guides** | `dsl/tutorial/guides/` | Task pages: install the CLI (points to `/install/`), use the Mac app (Predict, Break it, Compare), write a look, publish a package, use Synod with Claude or another agent |
| **Errors** | `synod explain --json` of the pinned compiler | One page per diagnostic code (`spec/11`): the message, a minimal program that triggers it, the fix |
| **Design** | `README.md` principles and `spec/decisions.md` | Why Synod is the way it is |
| **Tribute** | site | Lamport's "The Part-Time Parliament" and the name |

The Library and Errors pages are generated at site build time by running the
native pinned `synod` (`synod doc --json`, `synod explain --json`); the
WebAssembly API has neither command and is unchanged (decisions 90 and 93).

Rules for every section:

- Every `synod` block that starts with `# package` is a complete file.
  Every program on the site has a **Run** button: it calls `synod.wasm`
  (`run`, in a Web Worker) and plays the scene with `player` (a pinned
  release, assets verified against `SHA256SUMS`; the site supplies the fonts,
  as the player bundles none). A snippet
  that imports the examples is bundled with them under
  `vendor/github.com/synodlang/examples@<ver>/`. Diagnostics are shown inline, with a
  link to their Errors page.
- Docs are versioned by `dsl` tag (`/docs/v0.2/…`); the latest version is
  the default. Search is static (built at deploy time, no server).
- Pages link between sections: a tutorial chapter links to the reference
  section of each form it introduces, and each reference section links
  back to the chapter that teaches it.

## The tutorial

### Principles of the tutorial

The tutorial teaches the way the language was designed: from a tiny core
outwards, adding one idea at a time when a program needs it.

1. **Bottom up.** It starts with the smallest program that runs and
   shows something, and grows from there. No chapter uses syntax that has
   not been introduced.
2. **Syntax by need.** Each chapter opens with a goal the current
   program cannot reach ("two nodes should talk"). It then introduces
   exactly the syntax that closes the gap, and nothing else.
3. **Build on the previous piece.** Each chapter starts from the code
   that the previous chapter ended with, wherever the topic allows. A
   reader who types along keeps one growing module.
4. **Always runnable.** Every step is a complete program that passes
   `synod check` and runs in the page. The reader runs it, watches, then
   changes it.
5. **One way, shown once.** Synod has one spelling per meaning, so the
   tutorial never shows alternatives. When a reader would naturally try
   another spelling (no parentheses, `self`, a field without its node),
   a short "Why it looks like this" note shows the diagnostic they would
   get and links to the decision behind it.
6. **Watch, predict, break.** Each chapter ends with exercises: predict
   what the next run shows, then change one line and explain the
   difference.

### Chapter format

Every chapter has the same parts:

- **Goal:** one sentence saying what the program should do that it
  cannot do yet.
- **New in this chapter:** the syntax and phrases introduced, each
  linked to its reference section.
- **Steps:** a sequence of complete programs. Each step changes a few
  lines from the one before, and the changed lines are highlighted.
- **What to watch:** what the animation shows and why.
- **Why it looks like this** (optional, short): the principle or
  decision behind a form.
- **Exercises:** two to four, each with its expected outcome hidden behind
  a toggle.

### Module and folders

Chapters 1–3 use one folder `hello/` with `hello.synod` (`# package main`) and
no `synod.mod`. From chapter 4 the reader keeps one growing module, created by
`synod new tutorial --module example.com/tutorial`; step 2 of chapter 4 renames
the package folder to `counter` (the reason is the `bad-header` diagnostic
shown in that chapter). The packages are `counter` (chapter 4), `pingpong`
(5–8), `rumor` (9–11 and 13), `rumorlook` (12–13), `service` and `gossip`
(14), `checks` (14), `raft` and `raftlook` (15). Every step folder is the
whole module as the reader has it at that step.

### Chapters

The 15 chapters below are the ladder. A phrase or form is "new" in exactly one
chapter, and a step uses only items introduced in its chapter or an earlier one
(decision 104). The `std` packages and phrases are introduced the same way.

**Part I — One program, no nodes**

| # | Chapter | Goal | New in this chapter |
| --- | --- | --- | --- |
| 1 | `01-hello` | Show a sentence on the stage | file header `# package main`; comments; indentation blocks; `use`; `define main`; text values; type `text`; context `main`; `std/lesson` `lesson.Note`; canonical form and `synod fmt` (why `indent`: indentation) |
| 2 | `02-values` | Show a computed sentence | `set x to …` locals; number values; `plus`, `minus`, `times`, `divided by`; parentheses (one item per hole); list literals `{…}`; types `number` and `list of T`; `std/text` (`text.From`, `text.Join`); why `unused-name`: an unused name is an error |
| 3 | `03-time` | Show sentences at different moments, and ask the viewer | duration values `ms` and `s`; `after` blocks; type `time`; `std/sim` (`sim.StopAt`); `std/time` (`time.Now`); `lesson.Ask` and `lesson.Ask … choosing` (decision 82); the simulated and presentation clocks |

**Part II — Nodes and messages**

| # | Chapter | Goal | New in this chapter |
| --- | --- | --- | --- |
| 4 | `04-first-node` | Put a node on the stage that counts | `synod new` and `synod.mod`; a package folder and its header; `Node has` fields; capitals mean exported; `define [n: Node] start`; `.` members; types `Node` and `package`; context `node`; `time.Every`; `sim.Spawn`; `sim.Crash`; `demo.synod` calls its own package qualified; why `bad-header`: the package folder and its header. Step 1 is the output of `synod new` byte for byte |
| 5 | `05-messages` | Two nodes play ping-pong | record types with `has`; `message … with`, also qualified (`main` builds `pingpong.Start`); `when [n: Node] (net.Received …)` with bindings; `deliver`; types `node` and `record`; `std/net` (`net.Send`, `net.Received`, `net.Latency`); `sim.Nobody`, `sim.Delivered`; main sets `a.pingpong.Peer` |
| 6 | `06-decisions` | Stop after ten rounds | `if` and `else`; bool values; `is`, `is not`, `less than`, `more than`, `at least`, `at most`; `and`, `or`, `not`; type `bool`; why `negated-is`: `not (a is b)` is not canonical |
| 7 | `07-phrases` | Remove the repeated lines | `define` patterns with words and typed holes; receiver holes; `result`; type `msg`; value and action phrases; `define [n: Node] Reply [m: msg] to [p: node]` called `n.Reply …`; `define Pair [a: Node] and [b: Node]` called from main; `define … native` (prose only: it belongs to std, why `native-outside-std`); why `duplicate-name` |
| 8 | `08-many-nodes` | Broadcast to every peer and count replies | `{}` (empty) and how its type is found; `map of K to V` and writing `list of T` in holes; `item … of`; `set item k of … to …`; `is in` and `is not in`; `each x in …` and `each k, v in …`; type `value` (prose); `std/lists` (`Append`, `Extend`, `Without`, `Size`); `std/maps` (`Size`, `Keys`); `std/math` (`math.Floor`, in `Majority of`); `sim.Cluster`; why `ambiguous-empty` |
| 9 | `09-chance` | Spread a rumour to random peers | `std/random` (`Pick`, `TimeBetween`, `Chance`); `sim.Seed` and `--seed`; the same seed gives the same run |

**Part III — When things go wrong**

| # | Chapter | Goal | New in this chapter |
| --- | --- | --- | --- |
| 10 | `10-faults` | Keep the rumour spreading through crashes and loss | block holes, continuation lines and `run` blocks in your own phrases (`Retry … times` / `otherwise`); type `block`; `std/disk` (`Write`, `Read`); `sim.Restart` and `start` running again; `net.Loss`, `net.Partition … from …`, `net.Heal`, `net.DropNext`; why `block-last`: a block hole cannot come in the middle of a pattern |
| 11 | `11-invariants` | Say what must never happen, then break it | `always`; context `check`; `sim.NodesRunning`, `sim.IsUp`, `sim.Nodes`, `sim.Broken`; the four contexts side by side; why `check-effect`: a node phrase called from `always` |

**Part IV — Seeing it**

| # | Chapter | Goal | New in this chapter |
| --- | --- | --- | --- |
| 12 | `12-looks` | Draw the system your way | `when` on trace events and typed bindings; `draw … as … for …` and reactive blocks; attachments (`name of v`); color values; types `shape` and `color`; context `look`; `std/look` (`Card`, `Fill`, `Layout`, `Label`…), `std/theme`, `std/look/basic`; why `context`: setting a field in an observer |
| 13 | `13-story` | Make the important moment stand out | `emit … by n` and `sim.Emitted`; type `field`; `sim.Changed` with a `field` hole; `look.Pulse` and `look.Effect`; captions from `lesson.Note`; why `emit-for-other`: emitting for another node |

**Part V — Building real systems**

| # | Chapter | Goal | New in this chapter |
| --- | --- | --- | --- |
| 14 | `14-packages` | Build a service on top of the rumour package | `use … as`; `require`, `synod get` and `synod.sum`; running is transitive, so `n.rumor.Known` and `p.Node` accepted as `q.Node`; a package that only holds invariants (`checks/`); a local `gossip` that builds on `examples/gossip`; a `service` package over `rumor`; why `unneeded-alias`: `use … as` only for a collision |
| 15 | `15-raft` | Build `examples/raft` from an empty folder | Nothing new: seven steps, each adding one piece (roles and terms, a randomized election timeout, votes and majority, heartbeats, the "one leader per term" invariant, crashing the leader, a look); the last step equals the example |

The tutorial ends with "Where next": the examples, the reference, and
the Break it and Compare guides.

### Source file format

All paths are under `dsl/tutorial/` (decision 103).

- `README.md`: the index. `coverage.md`: the table `| Item | Shown as | Reference | Chapter |`.
  `versions.env`: `SYNOD_VERSION`, `SYNOD_LINUX_AMD64_SHA256`, `COMPILER_COMMIT`,
  `EXAMPLES_VERSION`, `EXAMPLES_COMMIT`.
- `NN-slug.md`: the chapter, prose only. Its headings are `# N. Title`, `## Goal`,
  `## New in this chapter` (only `<!-- new -->` items), `## Steps`, `## What to watch`,
  an optional `## Why it looks like this` and `## Exercises` (two to four
  `<details><summary>Expected</summary>…</details>`).
- Directives are HTML comments: `<!-- step K program=<folder> -->` under `## Steps`
  and `<!-- why <slug> -->` under the optional section. A synod fence in a
  chapter holds only text that parses, since the spec test reads every Markdown
  file of this repository.
- `code/NN-slug/stepK/`: step K as a complete folder (the whole module from
  chapter 4); `code/NN-slug/stepK.summary`: the golden of `synod run <program>
  --summary --style short`; `code/NN-slug/why-<slug>/` with a file `expect`
  holding one line, the diagnostic code. `guides/*.md`: install, mac-app,
  write-a-look, publish-a-package and agents.
- The site shows the changed lines of step K as the difference to step K−1; step
  1 is compared with the last step of the previous chapter.

### Coverage

The compiler holds the catalog of forms in `internal/forms` (not linked into
`synod` or `synod.wasm`): the forms of `spec/02` §2.2 (about 64, from `header`
to `divided-by`), the 16 types of `spec/04` §4.1 and §4.2, the four contexts of
`spec/05` §5.4 and the std packages. Its test checks the catalog against the
spec text: every production of `spec/02` §2.2 has a form, every quoted
terminal of §2.1 and §2.2 occurs in some form, and the type, context and
package ids equal the spec tables and the `## std/` headings of
`dsl/std/README.md` (decision 104).

`dsl/tutorial/coverage.md` has one row per catalog item, named `<kind>:<id>` (`form:`,
`type:`, `context:`, `package:`, because ids repeat across kinds), with its chapter, and
may add rows `phrase:<pattern>` for std phrases. A package row's reference is
`std/README.md § std/<p>`. The catalog marks each item as shown or as taught in
prose (`form:define-native` and `type:value` are taught in prose), and every shown
item whose row names chapter N is used in at least one step of chapter N. `internal/forms/cmd/doccheck`, run
as `go run ./internal/forms/cmd/doccheck -synod <bin> -tutorial <dsl>/tutorial
-library <dsl>/std/examples -examples <examples checkout> [-update]`, fails when:

- a catalog item has no row, or has two, or a row's Reference names no spec heading;
- a chapter or step directive is missing, or a step folder or program does not exist;
- a step fails `synod fmt --check`, `synod get` (when it has `synod.sum`) or
  `synod check`, or its `synod run <program> --summary --style short` differs from
  its golden (`-update` rewrites the goldens);
- a `why-<slug>` program does not fail with the code in `expect`;
- a chapter uses a form, type, context or std phrase before the chapter that
  introduces it, or no step of the chapter named in a shown item's row uses it;
- step 1 of chapter 4 differs from `synod new tutorial --module example.com/tutorial`,
  the last step of chapter 15 differs from `examples/raft` at `EXAMPLES_VERSION`
  (`raftlook` and `demo.synod` after replacing the examples module path by
  `example.com/tutorial/`), or a `require` line from chapter 14 on names another version.

### Library examples

`dsl/std/examples/<package>/<name>/` is one example: a folder with a main file
(an example may use several phrases). A folder without a main file is a helper
package that some example imports. An example folder that holds a file `expect`
with one diagnostic code fails its run with exactly that code (as `sim/failing`
shows `sim.Failed`), and an example folder without `expect` runs to its end. `doccheck` takes every exported std phrase,
events included, from `synod doc --json` and fails when none of the examples of its
package uses it, or when an example fails `fmt --check`, `check` or `run`, or runs differently from
its `expect` (decision 105). `dsl/std/examples/README.md`, `| Phrase | Examples |`, is generated
by `doccheck -update` and CI fails when it is stale; the Library page shows these
examples as runnable widgets.

### Sources and checks

- The site renders each step from its files; it never copies code into
  Markdown by hand, and `synod new` is the one source of chapter 4's first step.
- The tutorial is versioned with `dsl`: a language change is not done
  until the chapters, coverage, Library examples and golden summaries are
  updated (see `.github` `CONTRIBUTING.md`). The `docs` job of this repository's
  CI installs the pinned `synod`, checks out the compiler and the examples at
  their pinned versions and runs `doccheck`; the compiler's CI runs it against the
  pinned `dsl` tag.
