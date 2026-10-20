# 3. Phrases

A **phrase** is a sentence pattern of words and typed holes, defined once
and then used like a built-in. Everything except the language words is a
phrase, including the whole standard library.

```synod
define Send [m: msg] to [n: node]
  # body
```

## 3.1 The pattern rules

1. A pattern contains **at least one word**.
2. **No two inline holes are adjacent.** `[a: number] plus [b: number]` is fine; `[a: number] [b: number]` is not.
3. **Every hole is typed**: `[n: node]`, never `[n]`.
4. **Block holes come last on the line.** A hole of type `block` is filled by an indented block, not inline. A pattern may have several block holes; each one after the first is introduced by its own word on its own line (this is how `if … else …` is built):
   ```synod
   define Retry [n: number] times [b: block] otherwise [c: block]
   ```
   used as
   ```synod
   Retry 3 times
     net.Send request to server
   otherwise
     emit (message GaveUp with Tries 3)
   ```
5. The **first word** of a pattern is the phrase's **head**. It carries
   both the export capital and, outside the package, the qualifier:
   `net.Send m to n`, `card look.Fill theme.Accent`.
6. Reserved words (`spec/01`) cannot be heads, but may appear as later
   words: `Send [m: msg] to [n: node]`, `Majority of [n: number]`.

## 3.2 Names and uniqueness

- A phrase's **name** is its word skeleton with holes removed: `Send … to`.
- Within a package, every name is defined **once**. There is no
  overloading by hole types or by number of holes.
- Phrases, record types, and node state fields share one namespace per
  package. Record fields are scoped to their record type.
- A package must not define a phrase whose skeleton equals a language
  word's skeleton.

## 3.3 Resolution

Each sentence is resolved independently:

1. **Candidates**: language words; the current package's phrases (bare
   heads); for each qualified word `pkg.Head` in the sentence, the
   exported phrases of `pkg` with head `Head`.
2. **Match**: a pattern matches when the sentence has the same number of
   items, literal words agree position by position, and each hole's item
   has a compatible type (see `spec/04`). Because a hole always takes
   exactly one item, there is no partial or longest match.
3. **Result**: exactly one candidate must match. Zero is an
   `unknown-phrase` error; two is an `ambiguous-phrase` error that names
   both definitions.

A bare word in a hole position is a **name** (local, hole parameter,
bound event name, state field, one-word phrase, or package name), never a
literal word of the pattern. Names are resolved in this order and must
not shadow each other (shadowing is an error):

1. block locals and `each` names, 2. bound names from `when`, 3. hole
parameters of the enclosing `define`, 4. the current package's node
state fields, 5. the current package's one-word phrases, 6. imported
package names.

## 3.4 Results

A phrase either performs actions or produces a value.

- A **value phrase** gives its value with `result`. `result` ends the
  phrase. The compiler checks that every path through a value phrase
  reaches a `result`, and that all results have the same type.
- An **action phrase** has no `result`.
- A value phrase may also have effects (for example `NewRequest` in the
  Cassandra example updates a counter); the distinction is only about
  whether it gives a value.
- A value phrase used as a statement, or an action phrase used as an
  argument, is an error.
- `result` inside a block argument (for example inside a `time.Every`
  block) returns from the `define` that lexically contains it; if there is
  none, it is an error.

```synod
define Majority of [n: number]
  result ((math.Floor (n divided by 2)) plus 1)
```

## 3.5 Block holes

Inside a `define`, the name of a block hole used alone as a statement
**runs the block**. The block runs in the caller's context and sees the
caller's names as they were when the phrase was called.

```synod
define Every [d: time] [b: block]
  b
  after d
    Every d b
```

## 3.6 Native phrases

`define <pattern> native "<id>"` binds a pattern to a primitive
implemented by the runtime. Only packages under `std/` may declare native
phrases. The id is namespaced: `"net.send"`, `"look.tween"`. Natives
have the same pattern rules and the same resolution as ordinary phrases.

Native **event phrases** (used after `when`) are declared the same way
and are tagged by the runtime as either a *node event* or a *trace event*
(see `spec/05` and `spec/08`).
