# Compiler (repo `compiler`)

Go module `github.com/synodlang/compiler`. Go 1.23+. No cgo. Only the
standard library plus `golang.org/x/...` where needed.

## Layout

```
cmd/synod/          CLI entry (05-cli.md)
internal/source/    files, positions, diagnostics
internal/lex/       tokens, INDENT/DEDENT, paths, colors vs comments
internal/parse/     structure tree: File → Line → Sentence → Item, Block
internal/format/    canonical printer; fmt = parse → print
internal/load/      synod.mod, synod.sum, module cache, package graph
internal/resolve/   phrase tables, pattern matching, name binding
internal/check/     types, inference, contexts, exports, unused names
internal/ir/        resolved program: phrases, handlers, records, state
internal/sim/       runtime (02-runtime.md)
internal/natives/   one Go function per std native id
internal/scene/     scene builder (03-scene-builder.md)
internal/trace/     trace and scene encoders (04-formats.md)
internal/doc/       std catalog generator for agent-skills
testdata/           golden tests
```

## Stages

1. **Lex.** Track indentation with a stack; emit INDENT/DEDENT. A `#` is a
   comment when followed by a space or end of line, a color when followed
   by six hex digits, otherwise a `syntax` error. Paths are lexed only
   right after `use`.
2. **Parse** the structure grammar (`spec/02` §2.1) into a tree that keeps
   every token and comment with positions. The parser knows no phrases.
3. **Canonical check.** Print the tree with the canonical printer and
   compare to the source. Any difference is a `non-canonical` error
   pointing at the first differing line. `synod fmt` writes the printed
   form instead. Negation rewriting (`spec/10`) happens in the printer.
4. **Load.** Read `synod.mod`, resolve imports to package folders (module
   cache for remote ones, `std/` to the bundled std), build the import
   graph, reject cycles and `import-main`.
5. **Declare.** For each package collect: record types (`X has`), node
   state (`node has`), phrases (`define`), handlers (`when`), invariants
   (`always`), startup statements, and `main`. Check names, uniqueness,
   reserved words and pattern rules.
6. **Resolve.** For each sentence build candidates (`spec/03` §3.3) and
   match. Represent a pattern as a slice of `Literal(word)` and
   `Hole(name, type)`; matching is a position-by-position comparison of
   equal-length slices. Hole items that are words are resolved as names
   in the order of §3.3. Inside `when`, unknown bare words and record type
   names in event holes become bindings.
7. **Check.**
   - Types: unify local/state/record-field types from initial values and
     assignments; infer `{}` as list or map from usage across the package;
     check operators per `spec/04` §4.5; value vs action phrases; results
     on every path.
   - Contexts: compute the context of each handler, startup statement,
     `main`, `always` and observer; check every phrase body once per
     context it is reachable from (`spec/05` §5.4); report `context` and
     provable `foreign-state` errors.
   - Exports, unused names and imports, aliases.
8. **Lower to IR.** A tree of typed nodes with resolved phrase targets,
   ready to interpret.

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
  siblings.
- Every example in `examples` must check, run and match its golden trace.
