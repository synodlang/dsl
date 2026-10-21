# 4. Types and values

## 4.1 Value types

| Type | Literals | Notes |
| --- | --- | --- |
| `text` | `"leader"` | immutable |
| `number` | `0`, `2.5`, `-1` | one numeric type, 64-bit float |
| `bool` | `true`, `false` | |
| `time` | `150ms`, `2s` | a duration; also the type of `time.Now` (time since the run started) |

## 4.2 Structural types

| Type | What it is |
| --- | --- |
| `list of T` | ordered values of type `T`, `{a, b, c}`, indexed from 0 |
| `map of K to V` | keys of type `K` (`text`, `number`, `bool` or a node type) to values of type `V` |
| `node` | any node; identity only (`spec/05` §5.1); `sim.Nobody` is the node value meaning "no node" |
| `Node`, `p.Node` | a node running the current package, or package `p`, with that package's fields |
| `msg` | any record value (messages and events are records) |
| `<RecordType>` | a specific record type declared with `has`, e.g. `Prepare`, `api.Get` |
| `shape` | a drawn shape (look context only) |
| `color` | a named color (look context only) |
| `field` | a node field named as a value: `Role` in its own package, `raft.Role` elsewhere; only in `field` holes (`sim.Changed`) |
| `package` | an imported package, or the current package by its own name, used as a value: `sim.Cluster of 3 running raft` |
| `block` | an indented block passed to a block hole |
| `value` | any type; only in native patterns (`std/`), alone or inside `list of` and `map of` |

A record type name is a valid hole type, and `msg` accepts any record.
Types nest: `[ns: list of node]`, `[m: map of node to number]`, `list of list of node`. Bare `list` and `map` are not types.

## 4.3 Records

A record type is declared with `has`; each line gives a field and its
initial value:

```synod
Promise has
  set number to 0
  set prior to 0
  set priorValue to ""
```

A record value is built with `message … with`, naming fields (here in node code, where `n` is the receiver):

```synod
message Promise with number n.Promised, prior n.AcceptedBallot, priorValue n.AcceptedValue
```

- Fields not named keep their initial value. Naming a field with its initial value is allowed.
- Fields are read with `.`: `Promise.number`, `p.prior`. Record fields are never package-qualified: `api.Get.Key` reads `Key` of the record bound as `api.Get`, and `r.Key` reads it from a local `r` of type `api.Get`. A record field belongs to its record type, which already names the package.
- A field is read from a value whose record type is known at compile time. A `msg` value has no readable fields.
- Record values are **immutable**: `set` cannot target a record field (`bad-place`). Messages and events are both records.
- Capitalized fields are readable from other packages; lower-case fields only inside the declaring package.

`Node has` uses the same form to declare the fields of the package's node type (see `spec/05`).

## 4.4 Lists and maps

- `{}` is an empty list or an empty map. Its type (list or map, and its element types) comes from the type expected where it appears (the field or local it is assigned to, the hole it fills), and otherwise from how that place is used anywhere in the package: `std/lists` phrases imply a list; `std/maps` phrases and `set item k of x` with a non-number key imply a map; the items added fix the element types. When a native's result has the same type as one of its holes (as `disk.Read … or [d: value]` does), the type expected for the result is also expected for that hole, so `disk.Read "pages" on n or {}` takes the type of the field or local it is assigned to. If the compiler cannot tell, it is an `ambiguous-empty` error.
- Element types are known at compile time: in `when [n: Node] (net.Received Promise from acceptor)`, which binds the record `Promise`, `lists.Append Promise to n.promises` makes `n.promises` a `list of Promise`, and adding an item of another type is a `type-mismatch`.
- Element access: `item 0 of n.peers`, `item k of n.Known`. Reading a missing
  index or key is a run-time error; check with `is in` first.
- Update: `set item k of n.Known to v`. Lists and maps are values; `set item`
  replaces the place's value with an updated copy.
- Membership: `k is in n.Known` (map keys), `n is in n.peers` (list items).
- Iteration, shaped like Go's `range`:

```synod
each p in n.peers
  # p: item
each i, p in n.peers
  # i: index, p: item
each k in n.Known
  # k: key
each k, v in n.Known
  # k: key, v: value
```

Unused locals and `each` names are compile errors (as in Go), so only the
form you need compiles. Names bound by a `when` pattern and hole
parameters may go unused, like Go function parameters.
Maps iterate in ascending key order (nodes order by creation), so runs are deterministic.

## 4.5 Operators

| Operator | Operands → result |
| --- | --- |
| `plus`, `minus` | number, number → number · time, time → time |
| `times` | number, number → number · time, number → time · number, time → time |
| `divided by` | number, number → number · time, number → time · time, time → number |
| `less than`, `more than`, `at least`, `at most` | number or time pairs → bool |
| `is`, `is not` | two values, the type of one accepted as the type of the other (§4.7) → bool |
| `is in`, `is not in` | value, list or map → bool |
| `and`, `or` | bool, bool → bool |
| `not` | bool → bool |

Comparing two values whose types are not accepted as one another (§4.7) is a `type-mismatch` compile error. Division by zero is a run-time error.

## 4.6 Type inference

- Local, node-field and record-field types are inferred from their initial
  value and every assignment; they never change.
- Hole types are declared, element types included. Value phrase result types are inferred.
- There are no conversions except through phrases (`text.From`).

## 4.7 Accepted types

A value of type `S` is accepted where type `T` is expected when one of these holds:

- `S` is `T`;
- `S` is a node type and `T` is `node`;
- `S` is `p.Node`, `T` is `q.Node`, and running `p` runs `q` (`p` is `q` or uses it, directly or transitively; `spec/05` §5.1);
- `S` is a record type and `T` is `msg`;
- `S` is `list of A`, `T` is `list of B`, and `A` is accepted as `B`; `map of K1 to V1` is accepted as `map of K2 to V2` when `K1` is accepted as `K2` and `V1` as `V2`.

`{}` is accepted as any list or map type. `is` and `is not` compare two values when the type of one is accepted as the type of the other.
