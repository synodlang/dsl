# Documentation (site `synodlang.github.io`)

The site carries all Synod documentation. It is built from `dsl` and
`std` at build time, so it always matches a tagged specification.

## Sections

| Section | Source | What it is |
| --- | --- | --- |
| **Tutorial** | `dsl/tutorial/` | Learn Synod from nothing, bottom up (below) |
| **Reference** | `dsl/spec/` | The specification, one page per chapter, normative |
| **Library** | `dsl/std/README.md` and `synod doc --json` of the pinned compiler | One page per std package: every phrase with its pattern, result, contexts and a runnable example |
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

### Chapters

The module is created once, in chapter 4, and grows chapter by chapter.
Chapters 1–3 are single main files.

**Part I — One program, no nodes**

| # | Chapter | Goal | New in this chapter |
| --- | --- | --- | --- |
| 1 | Hello, Synod | Show a sentence on the stage | file header `# package main`; `use`; `define main`; indentation blocks; a phrase call `lesson.Note "…"`; text; comments; Run; canonical form and `synod fmt` |
| 2 | Values and names | Show a computed sentence | `set x to …` locals; numbers; `plus`, `minus`, `times`, `divided by`; one item per hole, so nested phrases are parenthesized; `text.From`; list literals `{…}` and `text.Join`; unused locals are errors |
| 3 | Time | Show sentences at different moments, and ask the viewer | durations `ms` and `s`; `after` blocks; `sim.StopAt`; `time.Now`; `lesson.Ask` and Predict; the simulated and presentation clocks |

**Part II — Nodes and messages**

| # | Chapter | Goal | New in this chapter |
| --- | --- | --- | --- |
| 4 | A first node | Put a node on the stage that counts | `synod new` and `synod.mod`; a library package folder and its header; `Node has` fields; capitals mean exported, so exported fields appear on the card; receivers and `define [n: Node] start`; `.` members (`n.Count`); `time.Every`; `demo.synod` uses its own package qualified; `sim.Spawn`, `sim.Cluster of … running …` |
| 5 | Messages | Two nodes play ping-pong | record types with `has`; `message … with`; `net.Send … from n to …`; `when [n: Node] (net.Received Ping from sender)` and its bindings; record fields `Ping.Count`; `sim.Nobody`; main sets another package's field `a.pingpong.Peer`; `deliver` and `sim.Delivered`; `net.Latency` |
| 6 | Decisions | Stop after ten rounds | `if`; `else` as a continuation line; `bool`; `is`, `is not`, `less than`, `more than`, `at least`, `at most`; `and`, `or`, `not`, and how `synod fmt` rewrites `not (…)` |
| 7 | Your own phrases | Remove the repeated lines | `define` patterns with words and typed holes; value phrases and `result`; action phrases; receiver phrases (`define [n: Node] Answer …`, called `n.Answer …`); phrases without a receiver called from main (`define Pair [a: Node] and [b: Node]`); one name, one definition |
| 8 | Many nodes | Broadcast to every peer and count replies | `list of T` and `map of K to V` in holes; `{}` and how its type is found; `item … of`; `set item k of … to …`; `is in`; `each x in …` and `each k, v in …`; `std/lists` (`Append`, `Extend`, `Without`, `Size`); `std/maps` |
| 9 | Chance and determinism | Spread a rumour to random peers | `random.Pick`, `random.TimeBetween`, `random.Chance`; `sim.Seed`; the same seed gives the same run |

**Part III — When things go wrong**

| # | Chapter | Goal | New in this chapter |
| --- | --- | --- | --- |
| 10 | Faults | Keep the rumour spreading through crashes and loss | `sim.Crash`, `sim.Restart`, `start` runs again on restart; `net.Loss`, `net.Partition … from …`, `net.Heal`, `net.DropNext`; `disk.Write … on n` and `disk.Read … on n or …` survive restarts; block holes and continuation lines in your own phrases (`Retry … times` / `otherwise`) |
| 11 | Invariants | Say what must never happen, then break it | `always`; check context and phrases without a receiver that read many nodes (`sim.NodesRunning`, `sim.IsUp`); `sim.Broken`; the four contexts side by side |

**Part IV — Seeing it**

| # | Chapter | Goal | New in this chapter |
| --- | --- | --- | --- |
| 12 | Looks | Draw the system your way | look packages and observers (`when` on trace events); typed bindings `when (sim.Added [n: rumor.Node])`; `draw … as … for …` and reactive blocks; `name of v` attachments; `std/look` shapes, `look.Fill … with …`, layouts; `std/theme`; color phrases with a color literal; observers are pure |
| 13 | Telling the story | Make the important moment stand out | package events: `emit … by n` and `sim.Emitted`; `look.Pulse`; `sim.Changed` with a `field` hole; captions from `lesson.Note` |

**Part V — Building real systems**

| # | Chapter | Goal | New in this chapter |
| --- | --- | --- | --- |
| 14 | Packages and composition | Build a service on top of the rumour package | `use` of another package in the same module and of remote modules; versions and `synod.sum`; running is transitive, so `n.rumor.Known` and `p.Node` accepted as `q.Node`; `use … as` on a name collision; packages that only hold looks or invariants |
| 15 | Capstone: Raft leader election | Build `examples/raft` from an empty file | Nothing new: each step adds one piece (roles and terms, a randomized election timeout, votes and majority, heartbeats, the "one leader per term" invariant, crashing the leader, a look), and the last step equals the example |

The tutorial ends with "Where next": the examples, the reference, and
the Break it and Compare guides.

### Coverage

`dsl/tutorial/coverage.md` lists every language form in `spec/02` §2.2,
every type in `spec/04`, every context in `spec/05`, and every std
package, each with the chapter that introduces it. Each must appear
exactly once as "new". CI fails when:

- the spec gains a form, type, context or std package that has no chapter;
- a chapter uses a form, type, context or std package before the chapter
  that introduces it.

### Sources and checks

- Chapters are `dsl/tutorial/NN-slug.md`. Every step's files are under
  `dsl/tutorial/code/NN-slug/stepK/`, with the same `synod.mod` the
  reader created in chapter 4, so what the reader types is what CI
  checks.
- The site renders each step from those files; it never copies code
  into Markdown by hand.
- CI (from M2 on) runs `synod fmt --check` and `synod check` on every
  step. From M3 on it also runs `synod run` and compares with a golden
  trace summary (`synod run --summary`), so "What to watch" never goes
  out of date.
- The tutorial is versioned with `dsl`: a language change is not done
  until the chapters, coverage and golden summaries are updated (see
  `.github` `CONTRIBUTING.md`).
