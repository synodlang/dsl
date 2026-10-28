# 13. Story

## Goal

Make the important moment stand out.

## New in this chapter

<!-- new -->

## Steps

The card turns amber when a node learns the rumor, but the eye misses it. The moment deserves a mark. The look cannot know the moment unless the behaviour says so, because the look may not guess; it only sees facts. A package can add its own facts with `emit`.

An event is a record, declared with `has` like a message. `emit (message Learned with Text t) by n` records it in the trace, as `sim.Emitted`, with the node that did it. It is only allowed in node code and the node must be the receiver. A look observes it like any fact: `when (sim.Emitted rumor.Learned by n)`. Exported record types, with a capital, are visible to other packages. Step 1 emits when a node learns the rumor, and the look makes its card pulse and glow with `look.Pulse` and `look.Effect "glow" on`:

<!-- step 1 program=rumor -->

Plain state changes need no event. Each change of an exported node field is already a fact, `sim.Changed`. The field it names is a `field` hole: a field written as a name, not read as a value. When the name is a field of a package, the observer filters on it. Step 2 gives a red outline to a node that gives up on a peer, by watching `rumor.GaveUp`. A bare new word in the field hole would instead bind any field, and `old` and `new` would be of type `value`:

<!-- step 2 program=rumor -->

The last piece of the story is told to the viewer. `lesson.Note`, which you used in chapter 1, adds a caption. In step 3 the demo explains each stage as it happens:

<!-- step 3 program=rumor -->

## What to watch

Every time a node learns the rumor its card pulses and glows. The nodes of the far side stay quiet while the network is split, and red outlines appear on the nodes that give up. When the network heals, the glow crosses over. The captions explain what you see: the split, the heal.

## Why it looks like this

An event says what happened to one node, so the node that emits it must be the one running the code, as with `net.Send` and `disk.Write`. Emitting for another node is refused:

<!-- why emit-for-other -->

`synod check` says `error[foreign-state]`: naming a node other than the receiver would let one node act for another.

## Exercises

1. Make the Learned event carry the node's name.
<details><summary>Expected</summary>

Add a field to `Learned` and set it in the `message` expression. The look reads it with `Learned.field`.

</details>

2. Remove `look.Effect "glow" on (card of n)`.
<details><summary>Expected</summary>

The card still pulses, without the glow.

</details>

3. Watch `rumor.Known` instead of `rumor.GaveUp`.
<details><summary>Expected</summary>

The red outline appears when a node learns the rumor, since `Known` changes at that moment.

</details>
