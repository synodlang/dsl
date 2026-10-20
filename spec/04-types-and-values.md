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
| `list` | ordered values, `{a, b, c}`, indexed from 0 |
| `map` | key → value; keys are `text`, `number`, `bool` or `node` |
| `node` | a simulated node; `sim.Nobody` is the node value meaning "no node" |
| `msg` | any record value (messages and events are records) |
| `<RecordType>` | a specific record type declared with `has`, e.g. `Prepare`, `api.Get` |
| `shape` | a drawn shape (look context only) |
| `color` | a named color (look context only) |
| `field` | a field name used as a value, e.g. in `sim.Changed` patterns |
| `package` | an imported package, or the current package by its own name, used as a value: `sim.Cluster of 3 running raft` |
| `block` | an indented block passed to a block hole |
| `value` | any value; allowed only in native patterns (`std/`) |

A record type name is a valid hole type, and `msg` accepts any record.

## 4.3 Records

A record type is declared with `has`; each line gives a field and its
initial value:

```synod
Promise has
  set number to 0
  set prior to 0
  set priorValue to ""
```

A record value is built with `message … with`, naming fields:

```synod
message Promise with number Promised, prior AcceptedBallot, priorValue AcceptedValue
```

- Fields not named keep their initial value. Naming a field with its initial value is allowed.
- Fields are read with `of`: `number of Promise`. Record fields are never
  qualified, even for types from other packages (`Key of api.Get`),
  because the record's type already says where the field comes from.
- When a value's record type is not known at compile time, `field of
  value` must name a field that exactly one record type in scope has;
  the type is checked again at run time.
- Record values are **immutable**. Messages and events are both records.
- Capitalized fields are readable from other packages; lower-case fields only inside the declaring package.

`node has` is special: it declares this package's **per-node state** (see `spec/05`).

## 4.4 Lists and maps

- `{}` is an empty list or an empty map. Which one is decided by the type
  expected where it appears (the field or local it is assigned to, the
  hole it fills), and otherwise by how that place is used anywhere in the
  package: `std/lists` phrases imply a list; `std/maps` phrases and
  `set item k of x` with a non-number key imply a map. If the compiler
  cannot tell, it is an `ambiguous-empty` error.
- Element types are inferred the same way (`lists.Append Promise to
  promises` makes `promises` a list of `Promise`). When they cannot be
  inferred they are checked at run time.
- Element access: `item 0 of peers`, `item k of Known`. Reading a missing
  index or key is a run-time error; check with `is in` first.
- Update: `set item k of Known to v`. Lists and maps are values; `set item`
  replaces the place's value with an updated copy.
- Membership: `k is in Known` (map keys), `n is in peers` (list items).
- Iteration, shaped like Go's `range`:

```synod
each n in peers
  # n: item
each i, n in peers
  # i: index, n: item
each k in Known
  # k: key
each k, v in Known
  # k: key, v: value
```

Unused locals and `each` names are compile errors (as in Go), so only the
form you need compiles. Names bound by a `when` pattern and hole
parameters may go unused, like Go function parameters.
Maps iterate in ascending key order (nodes order by creation), so runs are deterministic.

Element types inside lists and maps are checked at run time, not at compile time.

## 4.5 Operators

| Operator | Operands → result |
| --- | --- |
| `plus`, `minus` | number, number → number · time, time → time |
| `times` | number, number → number · time, number → time · number, time → time |
| `divided by` | number, number → number · time, number → time · time, time → number |
| `less than`, `more than`, `at least`, `at most` | number or time pairs → bool |
| `is`, `is not` | two values of the same type → bool |
| `is in`, `is not in` | value, list or map → bool |
| `and`, `or` | bool, bool → bool |
| `not` | bool → bool |

Comparing values of different types is a compile error when the types
are known, and a run-time error otherwise. Division by zero is a
run-time error.

## 4.6 Type inference

- Local, state and record-field types are inferred from their initial
  value and every assignment; they never change.
- Hole types are declared. Value phrase result types are inferred.
- There are no conversions except through phrases (`text.From`).
