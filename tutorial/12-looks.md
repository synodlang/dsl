# 12. Looks

## Goal

Draw the system your own way.

## New in this chapter

<!-- new -->

## Steps

So far the stage has been drawn for you, by a default **look**. A look is a package like the others, but it only watches: it reacts to the facts of the trace, draws, and never changes what happens. Nothing you write in a look can alter a run, so you can change how a system looks without ever risking what it does.

A look reacts with `when` followed by a **trace event**: `sim.Started`, `sim.Added n`, `net.Sent m from a to b`. Trace events have no receiver, because they happen to the whole system. The words after the event name new names for the parts of the fact (`n` is the node that was added), or, if a name already exists, filter on it.

Look code runs in the fourth context, **look**. It reads the exported state of nodes, as it was after that fact, and it has the drawing phrases of `std/look`. Using `std/look` also switches the default look off, so the package must draw the cards itself. We start a new package, `rumorlook`, and import it from the demo; a package with observers is active as soon as it is imported. Step 1 puts the nodes on a ring with `look.Layout "ring"` and draws one card for each node:

<!-- step 1 program=rumor -->

`draw (look.Card for n) as card for n` creates a shape, gives it the name `card`, and attaches it to the node `n` with `for n`. A shape has the type `shape`. Attached shapes can be reached later from any observer with `card of n`. Messages are invisible for now. Step 2 draws a ball per message: it takes off at `net.Sent`, flies between the two cards with `look.Flight`, and is removed when the message arrives. The ball is attached to the message id, `net.Id m`, so `ball of (net.Id m)` finds it later:

<!-- step 2 program=rumor -->

A card should show what the node knows. A `draw` can end with an indented block that is **reactive**: it runs again whenever the state it reads changes, and the card morphs to the new look. A binding with a type, `[n: rumor.Node]`, tells the compiler that `n` runs `rumor`, so `n.rumor.Known` is readable. Colors are values too, of type `color`, but only named ones: a color is a phrase with no holes whose `result` is the literal. `Heard` is ours, and `theme.Quiet` comes from `std/theme`:

<!-- step 3 program=rumor -->

That is a lot of code to redo what the default look already did. A look package may use `std/look/basic` to keep the default and change only what it wants. Step 4 keeps the basic look (cards, messages, captions) and replaces just the card with a tinted one:

<!-- step 4 program=rumor -->

Looks are the place for taste. The behaviour never changes: run the program without the look and you get the same trace.

## What to watch

Step 1: six empty cards on a ring and no messages. Step 2: balls travel between cards; a dropped message stops halfway. Step 3 and 4: a card is grey until its node learns the rumor, then it turns amber, smoothly. The partition is visible: the amber spreads on one side and waits on the other.

## Why it looks like this

A look cannot change what happens. An observer that tries to set a node field is refused:

<!-- why context -->

`synod check` says `error[context]`: setting state is not allowed in look context. This is what lets the trace be the same with and without a look.

## Exercises

1. Change the colors: make unknowing cards `theme.Critical`.
<details><summary>Expected</summary>

Use `theme.Critical` in `Tint`. The cards start red and turn amber.

</details>

2. In step 2 use `look.Layout "row"`.
<details><summary>Expected</summary>

The six cards stand in a row instead of a ring.

</details>

3. In step 4, remove the `use std/look/basic` line.
<details><summary>Expected</summary>

The program still checks, but messages are no longer drawn, because only the cards are drawn by this package.

</details>
