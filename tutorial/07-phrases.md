# 7. Phrases

## Goal

Remove the repeated lines from the game, by naming them.

## New in this chapter

<!-- new -->

## Steps

Look at the handlers of chapter 6: three times the same long line, `net.Send … from n to …`. A **phrase** gives a sentence a name, so that a program reads as the story it tells. You have used phrases from the first chapter: `lesson.Note`, `time.Every`, `net.Send` are phrases, written by someone else and imported. Now you write your own.

A phrase is defined with `define`, followed by its **pattern** and a block. The pattern is made of words and **holes**. A hole `[m: msg]` is a place where the caller puts one item; it has a name, `m`, usable in the block, and a type after the colon. Every hole is typed. `msg` is a new type: any record, any message. The first word of the pattern is the **head**; it begins with a capital because the phrase is exported.

When a hole comes before the head, `[n: Node]`, it is the **receiver**: the node the phrase works on. The call joins it to the head with a dot, like a field: `n.Reply …`. A phrase with a receiver is node code, and in its block `n` is the node itself.

Step 1 defines `Reply`: send a message from this node to another. The two handlers that answer use it:

<!-- step 1 program=pingpong -->

Read the handler again: `n.Reply (message Pong with round Ping.round) to sender`. It says what happens, and the words in between (`Reply`, `to`) are chosen by you. The pattern `define [n: Node] Reply [m: msg] to [p: node]` is a sentence with two holes and one literal word, `to`; a call has to have the same shape, word for word.

A phrase can also **give a value** with `result`. Step 2 names the limit of ten rounds, once. `Limit` has no holes, so it is one item, written bare, and a call is just the word:

<!-- step 2 program=pingpong -->

A phrase that has `result` is a **value phrase** and is used where a value goes, in an argument. A phrase without `result` is an **action phrase** and stands as a statement. Mixing them up is an error.

Phrases do not need a receiver. Step 3 defines `Pair`, with two holes of type `Node`, that makes two nodes each other's partner; the demo calls it as `pingpong.Pair a and b`, qualified like every phrase of another package:

<!-- step 3 program=pingpong -->

The program behaves exactly as before: the point of a refactoring is that nothing changes. Compare `step3.summary` with the summary of chapter 6.

One more form exists, which you will see and not write: a phrase whose body is not Synod but part of the simulator, `define Size [l: list of value] native "lists.size"`. The standard library is full of them. `native` is the only way a phrase gets power the language does not have, so only packages under `std/` may use it, and `value` is a type that only they may use in holes.

## What to watch

Nothing different on the stage: the same ten rounds. That is the check that you did a refactoring and not a change.

## Why it looks like this

A name is defined once per package. There is no overloading by hole types, by number of holes or by having a receiver; two phrases that read the same would make a program ambiguous. Define `Pair` twice with different holes:

<!-- why duplicate-name -->

`synod check` says `error[duplicate-name]` and points at both definitions.

And `native` outside the standard library is refused:

<!-- why native-outside-std -->

`error[native-outside-std]`: a phrase you write is always made of other phrases, which keeps every program readable down to the language.

## Exercises

1. Add a phrase `define [n: Node] Finish` that sets `Done` to true and use it in the two places.
<details><summary>Expected</summary>

Same run. The body is `set n.Done to true`; the calls are `n.Finish`. A receiver phrase with no other hole is one item, written bare.

</details>

2. Make `Limit` return 3.
<details><summary>Expected</summary>

One edit, three rounds, as in the exercise of chapter 6, which needed two.

</details>

3. Call `a.pingpong.Reply (message pingpong.Start with Round 1) to b` from `define main`. What happens?
<details><summary>Expected</summary>

`error[context]`: a receiver phrase is node code and `main` is not a node.

</details>
