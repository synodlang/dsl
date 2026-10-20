# 5. Nodes, state and contexts

## 5.1 Running a package

A node **runs** packages. `sim.Node running raft` creates a node that runs
`raft`; running a package also runs, transitively, every package it uses
that declares node state, startup statements or node handlers. So a node
running `cassandra` also runs `gossip` and `ring` if `cassandra` uses
them. Each run package is started once per node, in dependency order
(dependencies first).

## 5.2 Per-node state

```synod
node has
  set Heartbeat to 0
  set Known to {}
  set peers to {}
```

- Each package that declares `node has` gives every node running it its
  own copy of these fields, namespaced by package.
- Inside the package the current node's fields are bare names (`Heartbeat`).
  Another node's fields use `of` (`Heartbeat of n`). From other packages the
  field is qualified (`gossip.Heartbeat`, `gossip.Heartbeat of n`), and must
  be exported.
- There is no `self`: in node context the current node is implicit.
  `sim.Here` gives the current node as a value when one is needed.
- Every node also has identity fields from `std/sim`: `sim.Id` (a
  number, 1, 2, 3… in creation order) and `sim.Name` (`"n1"`, `"n2"`…).
  Identity is readable in every context.
- Reading a field of a node that does not run the declaring package is a
  run-time error.

## 5.3 Top-level statements

Top-level lines in a package are either declarations (`use`, `has`,
`define`, `when`, `always`) or **startup statements**. Startup statements
run on every node running the package, in file order (files in name
order), when the node starts and again on every restart:

```synod
time.Every 1s
  set Heartbeat to (Heartbeat plus 1)
```

## 5.4 The four contexts

Every statement runs in exactly one context, and the compiler checks
what each context may do.

| Context | Where | Reads | Writes | May use |
| --- | --- | --- | --- | --- |
| **node** | startup statements; `when` on node events; phrases called from them | own node's state; identity of any node | own node's state | `net.Send`, `after` (simulated), `emit`, `disk.*` |
| **main** | the `main` block and its `after` blocks | any node's state | any node's state (setup and faults) | `deliver`, `after` (simulated), `emit`, `sim.*` setup and fault phrases |
| **check** | `always` and phrases called from it | any node's state | locals only | nothing with effects |
| **look** | `when` on trace events; phrases called from them | exported state of any node, as it was at that trace event | locals and shapes only | `draw`, `std/look`, `after` (presentation) |

Rules that follow:

- **Realism.** A node handler cannot read or change another node's state;
  it must send a message. Violations that the compiler can prove are
  compile errors (`foreign-state`); others are run-time errors.
- **Purity of looks.** Observers cannot `set` node state, `deliver`,
  `emit`, or send. A look can never change what happens, only how it looks.
- **Phrases are context-polymorphic.** A phrase is checked for each
  context it is called from. A phrase reading bare state fields can only
  be called where there is a current node.

## 5.5 Node events vs trace events

`when` takes an event phrase. Event phrases are native and come in two kinds:

- **Node events** are delivered to one node and run a handler in node context:
  `net.Received`, `sim.Delivered`.
- **Trace events** are facts recorded in the trace and run observers in look
  context: `net.Sent`, `sim.Changed`, `sim.Emitted`… (full list in `spec/08`).

A `when` on a node event runs only on nodes running the package that
contains the `when`. A `when` on a trace event runs once per occurrence,
for every package in the program that contains it.

## 5.6 Matching node events

In `when (net.Received Prepare from proposer)`:

- A **record type name** in a `msg` hole filters by type and binds the
  record under that name (`number of Prepare`). Qualified types bind under
  the qualified name (`Key of api.Get`).
- A **new bare word** in any other hole binds the value (`proposer`).
- An **existing name or a literal** filters by equality.

## 5.7 Invariants

```synod
always ((lists.Size ChosenValues) at most 1)
```

`always` takes a `bool`. It is evaluated after every simulation step (see
`spec/07`) in check context. When it becomes false the trace records
`sim.Broken`, once per transition from true to false; the run continues.
