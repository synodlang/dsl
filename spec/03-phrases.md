# 3. Phrases

A **phrase** is a sentence pattern of words and typed holes, defined once
and then used like a built-in. Everything except the language words and forms is a
phrase, including the whole standard library.

```synod
define Send [m: msg] to [n: node]
  emit m by n
```

## 3.1 The pattern rules

1. A pattern contains **at least one word**. Its first word is the
   phrase's **head**.
2. **Receiver.** A pattern may start with one hole before its head, the
   **receiver**. Its type is a node type, `Node` or `p.Node`:
   ```synod
   define [n: Node] Learn [k: node] at [v: number]
   ```
   A phrase with a receiver is a **receiver phrase**. It is node-context
   code (`spec/05`) and is called with its receiver joined to its head by
   `.`: `n.Learn k at v`. Every other hole comes after the head.
3. **No two inline holes are adjacent.** `[a: number] plus [b: number]` is fine; `[a: number] [b: number]` is not.
4. **Every hole is typed**, element types included: `[n: node]`,
   `[ns: list of Node]`. Never `[n]` or `[ns: list]`.
5. **Block holes come last.** A hole of type `block` is filled by an
   indented block, not inline. At least one word or inline hole stands
   between the head and the first block hole. After a block hole come
   only further pairs of one word and one block hole; each such word
   starts a continuation line (`spec/02` §2.1). This is how `if … else`
   is built:
   ```synod
   define Retry [n: number] times [b: block] otherwise [c: block]
   ```
   used as
   ```synod
   Retry 3 times
     net.Send request from n to server
   otherwise
     emit (message GaveUp with Tries 3) by n
   ```
6. The head carries the export capital and, outside its package, the
   qualifier: `net.Send m to n`, `n.gossip.Learn k at v`,
   `look.Fill card with theme.Accent`.
7. Reserved words (`spec/01`) are never heads and may appear as later
   words: `Send [m: msg] to [n: node]`, `Majority of [n: number]`.
8. **A head is never directly followed by a comparison operator** (`is`,
   `is not`, `is in`, `is not in`, `less than`, `more than`, `at least`,
   `at most`): `Count is [n: number]` is a `bad-pattern` error, because
   its sentences read as comparisons and `not (…)` could not be
   rewritten without changing their meaning.

## 3.2 Names and uniqueness

- A phrase's **name** is its word skeleton, with the receiver and holes removed: `Learn … at`.
- Within a package every name is defined **once**. There is no
  overloading by hole types, by number of holes, or by having a receiver.
- One namespace per package holds its phrases, its record types, its
  node type `Node` and its node fields. Record fields are scoped to their
  record type.
- `Node` is the node type's name in every package, so no phrase head,
  record type or field in any package may be named `Node`
  (`duplicate-name`).
- A package must not define a phrase whose skeleton equals a language form's skeleton.

## 3.3 Resolution

Each sentence is resolved independently.

1. **Candidates** come from the sentence's first item:
   - the language forms (`spec/02` §2.2);
   - a bare word `H`: the current package's phrases without a receiver whose head is `H`;
   - a member `p.H`, where `p` is an imported package: `p`'s exported
     phrases without a receiver whose head is `H`;
   - a member `x.H` or `x.p.H`, where `x` names a node value: the receiver
     phrases with head `H` of the current package, or the exported ones of `p`.
2. **Match.** A pattern matches when the sentence has the same number of
   items, literal words agree position by position, and each hole's item
   has an accepted type (`spec/04` §4.7). For a receiver phrase the first
   item `x.H` (or `x.p.H`) stands for the receiver and the head together:
   `x` fills the receiver hole and the remaining items match the rest of
   the pattern. `n.Learn k at v` matches `[n: Node] Learn [k: node] at [v: number]`.
   Because a hole always takes exactly one item, there is no partial or
   longest match.
3. **Result.** Exactly one candidate must match. Zero is an
   `unknown-phrase` error; two is an `ambiguous-phrase` error that names
   both definitions.

### Names

A bare word in an argument position is a **name**, never a literal word
of the pattern. A name is exactly one of:

- a block local or an `each` name;
- a name bound by the enclosing `when`, its receiver included;
- a hole parameter of the enclosing `define`, its receiver included;
- a zero-hole phrase without a receiver of the current package;
- a package name: an import, or the current package's own name as a `package` value.

Two of these with the same spelling in scope is a `shadowing` error, so
every name has one meaning. Inside a `when` event phrase, a bare word that
is not yet a name binds a new one (`spec/05` §5.6). In a `field` hole, a
bare word names a node field of the current package; it does not read it.

### Members

A member is read from left to right. Its first word is a name. Each
following word is a named member of what comes before it:

| Before `.` | After `.` | Examples |
| --- | --- | --- |
| a package `p` | an exported phrase head, record type or `Node` of `p`; in a `field` hole, an exported node field of `p` | `net.Send`, `api.Get`, `gossip.Node`, `raft.Role` |
| a record value of type `R` | a field of `R` | `Vote.term`, `best.stamp`, `h.mutation` |
| a node value of type `Node` or `q.Node` | a node field or receiver phrase head of the current package; or a package `p` the node runs, then an exported node field or receiver phrase head of `p` | `n.Known`, `n.Version`, `n.gossip.Known`, `server.tcp.State` |

- A `when` that matches a qualified record type binds the record under
  that spelling, so `api.Get` names the record value and `api.Get.Key`
  reads its field.
- A value of type `node` has no members. `msg` values have no members.
- A member that ends at a field is a value, and can also be a place for `set`.
  A member that ends at a phrase head is a call.

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
- `result` appears only in phrase definitions; inside a block argument (for example a `time.Every` block) it returns from the `define` that lexically contains it. `result` in `define main`, `start`, `when` or `always` is a `context` error.

```synod
define Majority of [n: number]
  result ((math.Floor (n divided by 2)) plus 1)
```

## 3.5 Block holes

Inside a `define`, the name of a block hole used alone as a statement
**runs the block**. The block runs in the caller's context and sees the
caller's names as they were when the phrase was called. In node code the
block sees the receiver: inside `define [n: Node] start`, a `time.Every 1s`
block uses `n`.

```synod
define Every [d: time] [b: block]
  b
  after d
    Every d
      b
```

A block hole is filled only by an indented block, never by an argument:
the name `b` is never passed inline, and `Every d` above is filled with
a block that runs `b`.

## 3.6 Native phrases

`define <pattern> native "<id>"` binds a pattern to a primitive
implemented by the runtime. Only packages under `std/` may declare native
phrases. The id is namespaced: `"net.send"`, `"look.tween"`. Natives
have the same pattern rules and the same resolution as ordinary phrases.

Native **event phrases** (used after `when`) are declared the same way
and are tagged by the runtime as either a *node event* or a *trace event*
(see `spec/05` and `spec/08`).
