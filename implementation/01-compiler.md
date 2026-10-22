# Compiler (repo `compiler`)

Go module `github.com/synodlang/compiler`. Go 1.23+. No cgo. Only the
standard library plus `golang.org/x/...` where needed.

## Layout

```
cmd/synod/          CLI entry (05-cli.md)
cmd/synod-wasm/     WebAssembly entry: registers the global `synod` (see Targets)
internal/pipeline/  check/run/fmt output assembly shared by the CLI and wasm
internal/jsapi/     the JS API in pure Go (JSON in, plain values out)
internal/source/    files, positions, diagnostics
internal/lex/       tokens, INDENT/DEDENT, paths, colors vs comments
internal/parse/     structure tree: File → Line → Sentence → Item, Block
internal/format/    canonical printer; fmt = parse → print
internal/load/      synod.mod, synod.sum, vendor/ lookup, package graph (read-only: it never writes `vendor/` or the store)
internal/resolve/   phrase tables, pattern matching, name binding
internal/check/     types, inference, contexts, exports, unused names
internal/ir/        resolved program: phrases, handlers, records, state
internal/sim/       runtime (02-runtime.md)
internal/natives/   one Go function per std native id
internal/scene/     scene builder (03-scene-builder.md)
internal/trace/     trace and scene encoders (04-formats.md)
internal/doc/       `synod doc`: packages from source, short, rich and JSON forms, `--scope`, the std catalog for agent-skills
internal/explain/   `synod explain`: embedded entries `codes/<code>.md` (the entry format of decision 93), run in memory at explain time
internal/mcp/       `synod mcp`: the stdio JSON-RPC loop, dispatch and schema registry (stdlib only, no network package)
internal/get/       `synod get`: git download, the store `~/.synod/cache`, `synod.sum`, `vendor/` copies
testdata/           golden tests
```

## Stages

1. **Lex.** Track indentation with a stack; emit INDENT/DEDENT. A `#` is a
   comment when followed by a space or end of line, a color when followed
   by six hex digits, otherwise a `syntax` error. Paths are lexed only
   right after `use`. Members are one token, `word("."word)*`.
2. **Parse** the structure grammar (`spec/02` §2.1) into a tree that keeps
   every token and comment with positions. Commas split `items`;
   continuation lines are attached to the preceding statement (`spec/02`
   §2.1); holes are accepted in `define` and `when`. The parser knows no
   phrases.
3. **Canonical check.** Print the tree with the canonical printer and
   compare to the source. Any difference is a `non-canonical` error
   pointing at the first differing line. `synod fmt` writes the printed
   form instead. Negation rewriting (`spec/10`) happens in the printer.
4. **Load.** Read `synod.mod`, resolve imports to package folders (module
   the main module's `vendor/` for remote ones, which is the only place the compiler reads; `synod get` (`internal/get`) fills it from the download store `~/.synod/cache/<module>@<version>/`, verified against `synod.sum`, `std/` to the bundled std, which is authored in
   the std repo and vendored into the compiler with `embed`). Split each folder
   by header into the library package and at most one main file
   (`bad-header`, `extra-main`). Imports resolve to library packages
   (`unknown-import`); reject cycles. This replaces `import-main`.
5. **Declare.** For each package collect: record types (`X has`), the
   node type (every library package gets `Node`; `Node has` gives its
   fields), phrases with and without receivers (`define`), the `start`
   entry, `define main`, handlers (`when`; receiver versus event kind,
   `bad-receiver`) and invariants (`always`). Check names, uniqueness
   (one namespace per package, `spec/03` §3.2), reserved words and
   pattern rules.
6. **Resolve.** Resolve types (`list of`, `map of … to`, `p.Node`). For
   each sentence build candidates (`spec/03` §3.3) and match; members
   are resolved left to right, candidates come from the first item, and
   the receiver item fills the receiver hole. Represent a pattern as a
   slice of `Literal(word)` and `Hole(name, type)`; matching is a
   position-by-position comparison of equal-length slices. Hole items
   that are words are resolved as names in the order of §3.3. Inside
   `when`, unknown bare words, record type names and typed bindings
   (`[n: p.Node]`) in event holes become bindings.
7. **Check.**
   - Types: unify local/state/record-field types from initial values and
     assignments; check element types and node-type acceptance from the
     running relation (`spec/04` §4.7); infer `{}` from usage and the
     expected type; check operators per `spec/04` §4.5; value vs action
     phrases; results on every path; `bad-place`.
   - Contexts: contexts start at entries (`start`, `when`, `define main`,
     `always`, observers); check every phrase body once per context it is
     reachable from (`spec/05` §5.4); receiver phrases only in node
     context; report `context` errors. `foreign-state` is the lexical
     receiver rule and is always reported at compile time.
   - Exports, unused names and imports, aliases.
8. **Lower to IR.** A tree of typed nodes with resolved phrase targets,
   ready to interpret.

## Targets

- Native: `synod` for darwin (arm64 and amd64, shipped as one universal
  binary) and linux.
- WebAssembly: `GOOS=js GOARCH=wasm` build of `cmd/synod-wasm`, shipped as
  `synod-<tag>.wasm` with `synod-<tag>-wasm_exec.js` from the same Go
  toolchain. It sets a global `synod` with synchronous functions; the site
  runs the module in a Web Worker. Each returns a plain object and never
  throws (a Go panic or a JS exception through Go frames would kill the
  instance): `error` is null or `{code: "usage" | "internal", message}`;
  when set, `exitCode` is 2 and every other field is null.
  - `check(files, {paths?, tokens?, style?})` returns `{exitCode, error,
    check, text}`: `check` is the text of `synod check --json [--tokens]`,
    `text` that of `synod check --style S --color never`.
  - `run(files, {program, seed?, pace?, mode?, style?, summary?})` returns
    `{exitCode, error, check, status, summary, trace, scene, failure}`:
    `status` is the stdout of `synod run` with the default destinations
    (so it names `out/trace.jsonl` and `out/scene.json`), `summary` the
    `--summary` stdout or null, `trace` and `scene` the document texts
    (null when diagnostics stopped the run), `check` the `check.json` of
    those diagnostics or the ok document, `failure` null or `{kind: "run" |
    "look", line, diagnostic}`. Re-pacing is another call of `run`.
  - `format(files, {paths?, style?})` returns `{exitCode, error, files,
    changed, check, text}`: the canonical text of every selected file and
    the paths whose text differs.
  - `version()` returns `{api, language, compiler, formats, go, text}`.

  `files` maps relative slash paths to source text (no empty, absolute,
  `\`, `.` or `..` segments); the compiler reads only `synod.mod` and
  `*.synod` among them. Diagnostics show the paths as keyed, as if run from
  the tree root. std is embedded with `embed`; every other module is passed
  in `files` under `vendor/<module>@<version>/`, as `synod get` lays it
  out, with a `require` line in `synod.mod`, so memory and disk share one
  loader. Options are checked strictly: an unknown option, a bad `seed`
  or `pace` and a missing `program` are usage errors with the CLI's rules.
  Documents and texts are byte-identical to the CLI's. A size budget is
  enforced by the build tool and CI.

## Determinism across targets

Force float64 rounding with explicit conversions so that no fused
multiply-add happens on any `GOARCH`. CI runs the `runtest` and `driver` goldens
compiled to wasm under Node, diffs the trace hashes against Linux, and runs
a test that compares the JS API with the native results (trace, scene,
check, status and summary text) byte for byte.

## Fixes that need the package loaded

Settings order, the Names row and an own-package qualifier are
`non-canonical` errors that only the resolver can find. It records
`format.Fix` entries (renames, qualifier drops, setting reorders) that
`synod fmt` and the canonical check apply when the package loads;
otherwise `fmt` leaves them alone. There is still at most one
`non-canonical` diagnostic per file, with the canonical line as the hint.

## Diagnostics

All errors use `spec/11` codes. Collect as many as possible per run (stop
a package after its first parse error). Hints: for `unknown-phrase`,
suggest the closest pattern by edit distance over skeletons; for
`non-canonical`, show the canonical line.

## Tests

- **Golden tests** in `testdata/`: each case is a folder with sources and
  an expected `check.json`, `trace.jsonl` and `scene.json`.
- **Round trip:** for every canonical file, `fmt` is the identity.
- **Fuzz** the lexer and parser (`go test -fuzz`).
- **Spec snippets:** a test extracts every ```` ```synod ```` block from
  `dsl` that starts with `# package` and type-checks it together with its
  siblings; `# package main` blocks are grouped as standalone packages.
- Every example in `examples` must check, run and match its golden trace.
