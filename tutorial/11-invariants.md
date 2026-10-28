# 11. Invariants

## Goal

Say what must never happen, then make it happen.

## New in this chapter

<!-- new -->

## Steps

The animation shows what the system does, but you also want to say what it must never do. A rule written once and checked after every step of the run is an **invariant**, and Synod writes it with `always`.

`always` takes a condition and is checked after each step of the simulation. When the condition turns false, the trace records `sim.Broken` with the rule's source line, once, and the run goes on. A rule is usually a phrase that gives a `bool`, so it can be long and still read in one line: `always Agree`.

`Agree` looks at the whole system. It goes through `sim.NodesRunning rumor`, the list of running nodes of the package, and collects the different rumors that they know. The result is true when at most one differs. Reading another node's `Known` is fine here: this code runs in a new context, **check**, which may read the state of any node but can write nothing except its own locals. Step 1 adds the rule, and the run keeps it:

<!-- step 1 program=rumor -->

To break it, two rumors must start while the network is split. Step 2 asks the viewer to predict, then delivers a second rumor to the far side. The two halves end up knowing different rumors, and the trace shows `sim.Broken`:

<!-- step 2 program=rumor -->

Another rule counts nodes that are down. `sim.Nodes` is the list of all nodes, up or not, and `sim.IsUp n` says whether one is running; the rule says fewer than three may be down at any moment. Step 3 crashes three nodes at once and the rule breaks, but the others still spread the rumor:

<!-- step 3 program=rumor -->

You have now met all four contexts, and each is where it is for a reason:

| Context | Starts at | Reads | Writes |
| --- | --- | --- | --- |
| main | `define main` | any node's state | any node's state, and faults |
| node | `start`, `when [n: Node]`, receiver phrases | its own node | its own node |
| check | `always` | any node's state | only locals |
| look | `when` on trace events | exported state of any node | only shapes |

Look comes next chapter.

## What to watch

In step 2 the run goes on after the break: nothing stops, nothing is repaired. On the stage the border of the scene flashes red at the moment of `sim.Broken`. In step 3 the rule breaks as soon as the third node goes down, and only then.

## Why it looks like this

A rule is checked after every step, in any order, so it must have no effects. Calling a node phrase from `always` would send messages from the checker:

<!-- why check-effect -->

`synod check` says `error[context]` and the hint names the rule that calls it.

## Exercises

1. In step 2 deliver the second rumor to `item 0 of nodes` instead of `item 5`. Is the rule broken?
<details><summary>Expected</summary>

No. The first node already knows the first rumor, and a node believes only the first rumor it hears, so nothing differs.

</details>

2. Change `Down less than 3` to `Down less than 4`. What does step 3 show?
<details><summary>Expected</summary>

Three crashed nodes are allowed; `sim.Broken` does not appear.

</details>

3. Write a rule that at least four nodes are running.
<details><summary>Expected</summary>

`always ((lists.Size (sim.NodesRunning rumor)) at least 4)`, with `use std/lists`. Crash three nodes and it breaks.

</details>
