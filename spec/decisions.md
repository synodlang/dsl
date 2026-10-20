# Design decisions

Why Synod is the way it is. Add an entry whenever an implementation has
to decide something the spec did not.

| # | Decision | Why |
| --- | --- | --- |
| 1 | Consistency beats readability | A language people and agents write by imitation must look the same everywhere |
| 2 | One spelling per meaning; the formatter enforces it | Repos stay uniform; LLM output needs no style review |
| 3 | Everything is a phrase, including the standard library | A tiny kernel; the language grows by packages, not by keywords |
| 4 | A hole takes exactly one item; nested phrases are parenthesized | Exact matching, no precedence, no ambiguity from free-form sentences |
| 5 | Always-qualified imports, bare names inside a package (Go) | You can always see where a phrase comes from |
| 6 | Capitalization means exported, nothing else | One meaning per signal |
| 7 | `use … as` only on a last-segment collision | No second spelling of a package |
| 8 | No `self`; the current node is implicit; `sim.Here` when a value is needed | Less noise; identity stays available |
| 9 | `result` gives a phrase's value | Packages can define value helpers, not just natives |
| 10 | Collections iterate like Go's `range`; unused names are errors | Familiar, and only one loop form compiles |
| 11 | One number type, plus `time` with units | Fewer primitives; units stop `after 5` being ambiguous |
| 12 | Messages and events are immutable records declared with `has` | One declaration form for node state, messages and events |
| 13 | `{}` is an empty list or map, inferred from use | Accepted trade-off; unresolved cases are an error |
| 14 | `emit` lets packages add trace facts | Package concepts (leader elected) can animate richly |
| 15 | Behaviour is simulated, never narrated by an LLM | Faithful learning; Break it and Compare need real models |
| 16 | Two clocks: simulated and presentation | A 1ms hop can still animate over 400ms |
| 17 | Observers are pure | A look can never change what happens |
| 18 | Node handlers cannot touch other nodes' state | Realism: nodes only learn through messages |
| 19 | One motion curve, no easing parameter | Every package moves alike |
| 20 | Packages from Git URLs only, no registry | Go's model; nothing central to run |
| 21 | A package with `main` cannot be imported | Libraries and programs never mix |
| 22 | Behaviour runs on nodes by "running" a package; running is transitive over `use` | Roles (client, cache, db) and composition (Cassandra = gossip + ring + …) with no extra keyword |
| 23 | `deliver` is main-only; it injects outside requests | Clean boundary between the world and the cluster |
| 24 | `is not`, `at least` etc. exist, and the formatter rewrites `not (…)` into them | Readable operators without two spellings |
| 25 | Compiler, simulator and scene builder in Go; the Mac app only renders scenes | One portable implementation; a thin, fast native player |
| 26 | Records' fields are scoped per type; node fields share the package namespace | Several messages can all have a `term` field |
| 27 | `sim.Nobody` is the only "no value" | No general null; the one case that needs it (nodes) is explicit |
| 28 | Zero-hole one-word phrases are written bare | `time.Now`, `theme.Accent` read like names |
| 29 | `when` bindings and hole parameters may go unused; locals may not | Like Go parameters; no `_` placeholder needed |
| 30 | Importing a look or check package activates it without a reference | Looks are added by import, like Go's side-effect imports, without a `_` form |
| 31 | Node state is initialized at creation; startup statements run as the first event | `main` can adjust state (a server in LISTEN) before the node starts |
| 32 | Record fields are never qualified | The record's type already names the package |
