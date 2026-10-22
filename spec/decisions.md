# Design decisions

Why Synod is the way it is. Every decision holds on its own scope; none
overrides another. Add an entry whenever an implementation has to decide
something the spec did not.

| # | Decision | Why |
| --- | --- | --- |
| 1 | The design principles hold together; a design that cannot satisfy all of them is changed | No principle is traded against another, so rules never contradict |
| 2 | One spelling per meaning; the formatter enforces it | Repos stay uniform; LLM output needs no style review |
| 3 | The language is a fixed set of words and forms; everything else, the standard library included, is phrases | A tiny kernel; the language grows by packages, not by keywords |
| 4 | An inline hole takes exactly one item and a block hole one block; nested sentences are parenthesized | Exact matching, no precedence, no ambiguity from free-form sentences |
| 5 | Names from another package's namespace (its phrases, record types, `Node` and node fields) are always qualified; the current package's never are (Go) | You can always see where a name comes from |
| 6 | Capitalization means exported, nothing else | One meaning per signal |
| 7 | `use … as` only on a last-segment collision: two imports of a file share one, or an import's last segment is the importing package's own name | No second spelling of a package; a package `raft` can still import another `raft` |
| 8 | Node code names its node with a receiver, a node-typed hole before the head: `define [n: Node] Version`, `when [n: Node] (…)`, called `n.Version`; only node types are receivers | No implicit current node and no hidden state; every access shows whose state it is (Go receivers) |
| 9 | `result` gives a phrase's value | Packages can define value helpers, not just natives |
| 10 | Collections iterate like Go's `range`; unused locals and `each` names are errors | Familiar, and only one loop form compiles |
| 11 | One number type, plus `time` with units | Fewer primitives; units stop `after 5` being ambiguous |
| 12 | Node state, messages and events are declared with `has`; messages and events are immutable records | One declaration form for all three |
| 13 | `{}` is the one empty literal for lists and maps; its type comes from its use, and a `{}` whose type cannot be found is an `ambiguous-empty` error | One empty literal, never a guess |
| 14 | `emit` lets packages add trace facts | Package concepts (leader elected) can animate richly |
| 15 | Behaviour is simulated, never narrated by an LLM | Faithful learning; Break it and Compare need real models |
| 16 | Two clocks: simulated and presentation | A 1ms hop can still animate over 400ms |
| 17 | Observers are pure | A look can never change what happens |
| 18 | Node code reads and writes only its receiver's state; it reaches other nodes with `net.Send` | Realism: nodes only learn through messages |
| 19 | Node effects name their node: `net.Send m from n to p`, `disk.Write v at k on n`, `emit e by n`, spelled like the trace facts they record | Nothing acts for a node it does not name |
| 20 | One motion curve, no easing parameter | Every package moves alike |
| 21 | Packages from Git URLs only, no registry | Go's model; nothing central to run |
| 22 | A folder holds one library package and at most one main file (`# package main`, with `define main`); importing the folder never includes the main file | A library ships its demo next to it; libraries and programs never mix in one package |
| 23 | Behaviour runs on nodes by "running" a package; running is transitive over `use` | Roles (client, cache, db) and composition (Cassandra = gossip + ring + …) with no extra keyword |
| 24 | `deliver` is main-only; it injects outside requests | Clean boundary between the world and the cluster |
| 25 | `is not`, `at least` etc. exist, and the formatter rewrites `not (…)` into them | Readable operators without two spellings |
| 26 | Compiler, simulator and scene builder are written in Go and built natively and to WebAssembly; one JavaScript canvas library, `player`, draws scenes in the Mac app and on the site | One implementation of each stage, everywhere; the app stays a thin host |
| 27 | Node fields, phrases, record types and `Node` share one namespace per package; record fields belong to their record type, so they are never package-qualified | `n.Known` and `n.Version` never collide; several messages can all have a `term` field; a record's type already names its package |
| 28 | `sim.Nobody` is the only "no value" | No general null; the one case that needs it (nodes) is explicit |
| 29 | A zero-hole phrase is one item, written bare: `time.Now`, `theme.Accent`, `n.Version` | They read like names |
| 30 | `when` bindings and hole parameters, receivers included, may go unused | Like Go parameters; no `_` placeholder needed |
| 31 | Importing a package that has observers or invariants uses it | Looks are added by import, like Go's side-effect imports, without a `_` form |
| 32 | Node state is initialized when the node is created; `start` runs as the node's first event and again on every restart | `main` can adjust state (a server in LISTEN) before the node starts |
| 33 | Entry points are definitions: `define main` and `define [n: Node] start`; `main` and `start` are reserved for them; a file root holds only declarations | One definition form; nothing runs implicitly at file root |
| 34 | `.` reaches named members only: package members, record fields, node fields, receiver phrases; elements are `item … of`, attached shapes `name of v` | One meaning for `.`, one element form |
| 35 | Collection types name their element types, `list of T` and `map of K to V`, and are checked at compile time | Holes say exactly what they take; no run-time element errors |
| 36 | `node` is any node, identity only; every library package has a node type `Node` (`p.Node` outside) with its fields; a `p.Node` is accepted as `q.Node` when running `p` runs `q`, and as `node` | Fields are reachable only where the type proves they exist |
| 37 | A `when` event phrase may bind a node with a type, `[n: raft.Node]`, and then matches only such nodes | Observers read the fields of the nodes they draw |
| 38 | `lists.Append` adds one item; `lists.Extend` joins two lists | Two meanings, two phrases; no overloading |
| 39 | Commas stand only between list items, between `message … with` settings and between the two `each` names; a comma anywhere else is a `syntax` error | Every comma has one job, so a stray comma is caught at parse time |
| 40 | A continuation line (one word with a block, at its statement's indentation, directly after the statement's block) continues that statement; `else` is one | One rule builds `if … else` and every multi-block phrase |
| 41 | `net.DropNext` names the record type as text: `net.DropNext "Accept" from a to b` | One spelling in the catalog, the spec and Break it |
| 42 | The organization site is `synodlang.github.io`: static, with examples edited and re-run in the browser by `synod.wasm` | GitHub Pages for the organization; no server; interactive examples replace recordings |
| 43 | The header is `# package <word>` on the first non-blank line; with imports, one blank line follows the last import; without, the first declaration follows the header directly | One layout per file; the header and imports read as the file's preamble |
| 44 | Comments attach to lines (the next line, or the line they trail) and never form blocks or statements; a comment line's indentation is not checked | Comments cannot change structure; the formatter re-indents them |
| 45 | `+1`, `.5`, `x-1` and spaces around `.` are `syntax` errors, not formatter fixes | Each has no single obvious meaning to rewrite to, so the formatter does not guess |
| 46 | CR LF line endings and a byte order mark are `non-canonical`; `synod fmt` fixes them | They are harmless to read, so they are repaired, not rejected |
| 47 | Two atoms (words, members, numbers, durations, texts, colors) are always separated by whitespace | `x-1` is neither `x minus 1` nor the number `-1` after `x`; no silent guess |
| 48 | Imports are sorted by the byte order of the full path text, `@version` included; a `use` below a declaration is moved up | A total order that needs no locale or parsing; one place for imports |
| 49 | Blank and comment lines may sit between a block and its continuation line; the formatter drops the blanks and keeps the comments before the continuation | A comment never breaks `if … else` |
| 50 | The formatter rewrites `not (c)` into the direct operator both ways, `is not` into `is` included; `not (a and b)` stays; a pattern whose head is directly followed by a comparison operator is a `bad-pattern` error | One spelling per meaning in both directions; such a pattern would read as a comparison |
| 51 | The formatter removes parentheses but never inserts them; a missing pair is an `unknown-phrase` error | Wrapping would guess where an argument ends |
| 52 | A sentence's phrase is chosen by its shape (item count, literal words, block structure); types never select between phrases, and a type failure is `type-mismatch` | One reading per sentence; no overloading by type, and errors name the phrase you meant |
| 53 | `result` stands only where the phrase is still running: not in an `after` block or a block argument | Those blocks run after the phrase has returned, so there is nothing to return to |
| 54 | A local from outside an `after` block or block argument cannot be set inside it | The block holds a copy, so the set would silently do nothing |
| 55 | A local created in a block is visible only to the end of that block | Go's scoping; no value depends on which branch ran |
| 56 | Look context reads only exported state, even of its own package | Looks see what other packages see, and the trace records only that |
| 57 | The Context column of the std catalog is enforced at every call of a std phrase, written in Synod or native | `time.Every` cannot leak into look context just because its body would allow it |
| 58 | `draw … as w` declares a shape local; drawing `w` again in the same scope rebinds it; a draw name may go unused | Drawing is itself the effect, like decision 30; redrawing is not shadowing |
| 59 | A place's type is the join of its initial value and every assignment; unrelated node types do not join | One type per place, decided without guessing `node` |
| 60 | The compiler reads downloaded modules only from `vendor/<module path>@<version>/` at the root of the main module, ignored by git; `synod get` fills it from a shared download store `~/.synod/cache/<module>@<version>/` (verified against `synod.sum`, never edited), downloading only what the store lacks; no override | Dependencies live with the project and have one place to read; the store only saves repeated downloads, and `synod.sum` keeps fresh downloads reproducible |
| 61 | Every Synod repository shares MAJOR.MINOR with the language version and numbers its own PATCH: a `dsl` patch is a clarification or spec bug fix; a patch elsewhere is a bug fix or a feature implementing more of the same language version; a new language feature raises MINOR, spec first | One number says which language a release speaks; patches never change the language |
| 62 | The run's seed is `--seed`, else `sim.Seed`, else 1 | A command-line seed can replay any program, including demos that call `sim.Seed` |
| 63 | A message flight is the native `look.Flight … for m`; it spans the presentation slots of `net.Sent` and its landing fact; `net.Id` gives the id carried by a message bound by a net fact; equality ignores the id | Flights readable without look-ahead or animating in the past (rule 1 of `spec/09` §9.6) |
| 64 | Deleting a shape is the unexported `look.delete`; `look.Remove` is the one public spelling; a deleted shape stays attached; every look phrase on it does nothing | One spelling (decision 2); a late `card of n` never fails because a node left |
| 65 | For `"x"` and `"y"`, `look.Tween` takes a number or a shape; looks never read coordinates | `Travel` in Synod without exposing positions |
| 66 | Presentation time is whole ms and pace is in thousandths; every look duration (`after`, `Tween`, builder constants) is scaled by pace | Integer arithmetic, byte-identical scenes everywhere |
| 67 | A tween on a property supersedes the part of a running tween on that property after its start | One timeline per property, no queues |
| 68 | A run-time error in look context stops the scene, never the simulation; it is not recorded in the trace | Observers are pure (decision 17) |
| 69 | `look.Place` is a constraint: the shape follows its anchor until the shape itself is moved | Rows and tags follow cards when layouts re-flow |
| 70 | `synod.wasm` exposes a synchronous global `synod` (`check`, `run`, `format`, `version`); results are the CLI's bytes as strings and it never throws (errors are `{code, message}`, exit code 2) | Byte-identical to the CLI; exceptions cannot cross Go frames; a Web Worker keeps pages responsive |
| 71 | `synod.wasm` embeds only std; other modules come in `files`, under `vendor/` as `synod get` lays them out; the site copies `synod.wasm` and its matching `wasm_exec.js` from a pinned release at build time | One loader for disk and memory; examples stay editable; no CORS on release assets |
| 72 | In 0.2 every label is drawn in the player's sans at 16 px, weight 500, centered; size, weight and mono are not yet choosable and will be additive label props | The format has no text size or weight; adding props later breaks no reader (format rule: keys only added) |
| 73 | Readers draw `scene.json` by the fixed rules of `implementation/04-formats.md` (Drawing a scene); a frame test compares display lists, pixels are a secondary check | The same scene looks the same on every engine, and determinism can be tested exactly |
