# 10. Faults

## Goal

Keep the rumor spreading when nodes crash and messages are lost.

## New in this chapter

<!-- new -->

## Steps

Everything so far worked because nothing broke. Real nodes crash and come back, and real networks lose messages. This chapter takes the `rumor` package of chapter 9 and makes it survive. First we need a place for what must outlive a crash.

When a node restarts, every field goes back to its initial value. What a node writes to its **disk** stays. `std/disk` has two phrases: `disk.Write value at "key" on n` stores a value, and `disk.Read "key" on n or default` reads it back, or gives `default` when nothing is stored. Both name the node, which must be the one running the code.

The code has to use it at the right moment: `define [n: Node] start` runs when a node is created, and again after every restart, so that is where a node reloads what it kept. In step 1 a node writes the rumor when it learns it and reloads it in `start`. Peers have to be on disk too, or a restarted node would have nobody to talk to, so `Connect` now sends each node a `Join` record to store them. The demo crashes `n3` at 2s and restarts it, with `sim.Restart`, at 3s:

<!-- step 1 program=rumor -->

Look at `n3` in the summary after the restart: the fields were reset, then `start` read the disk and the node knew the rumor again. In chapter 9 some nodes never heard it at all, because nodes told a fixed number of times at random. Real systems confirm. Step 2 adds an **acknowledgement**: a node that receives the rumor answers with an `Ack`, and the teller keeps a list `Acked` of peers that answered. It tells only peers that are still `Pending`, and keeps going until there are none:

<!-- step 2 program=rumor -->

Now every node ends up knowing the rumor, even with seed 4. Next we break the network. `net.Loss 0.3` makes the network lose 30% of all messages, at random. A lost rumor is never acknowledged, so the teller should try again, a few times, and then give up for now. "Try this, and if it does not work in time, do that" is a pattern we will want in many places, so we make it a phrase, with a new kind of hole.

A hole of type `block` is filled with an indented block of statements instead of an item. Block holes come last, after at least one word or inline hole. After a block hole come pairs of one word and one more block hole. The word starts a **continuation line**: a line with a single word and a block, right after the block before it, at the same indentation. You know one, `else`; ours is `otherwise`. In the definition, the name of a block hole used as a statement **runs** the block: `b` runs the first, `c` the second. Look at the definition of `Retry`: it runs `b`, waits 300ms, and if the peer has still not answered it calls itself with the same two blocks until the tries are used up, then runs `c`.

The call reads like a sentence: `n.Retry 3 times for peer`, a block that sends, `otherwise`, a block that counts the failure:

<!-- step 3 program=rumor -->

The two blocks run in the caller's context and see its names, `n` and `peer`, as they were when `Retry` was called. They hold copies, so they cannot set a local of the caller; that is why the failure is recorded in a node field, `GaveUp`.

Last, the faults you aim. `net.Partition left from right` cuts every link between two groups of nodes until `net.Heal`. `net.DropNext "Rumor" from a to b` drops the next message of that record type on that link. Step 4 drops the first rumor from `n1` to `n2` and splits the cluster for almost three seconds:

<!-- step 4 program=rumor -->

## What to watch

In step 1, `n3` greys out at 2s and returns at 3s with the rumor already in its card. In step 3, some messages fade halfway: those are the lost ones; a card retells the same peer a moment later. In step 4 the rumor stays on the left side of the split. The right-hand nodes are retried and given up on, again and again, and as soon as the network heals the rumor crosses and the run ends. The retry never stops by itself: it is the `Pending` list that ends it.

## Why it looks like this

A block hole is filled by an indented block, so it cannot come in the middle of a pattern: after it only pairs of a word and a block may follow, and before it a word or an inline hole must stand. This pattern breaks both rules:

<!-- why block-last -->

`synod check` says `error[bad-pattern]`. The same rules make every call unambiguous, because the compiler always knows where a block begins and which word starts the next one.

## Exercises

1. In step 3 change `net.Loss 0.3` to `0.6`. What changes?
<details><summary>Expected</summary>

More messages are lost, more nodes give up, and `GaveUp` counts grow. The rumor still arrives, because the telling goes on until every peer acknowledges.

</details>

2. In step 1, restart `n3` at 2500ms instead of 3s. Does anything change in what it knows?
<details><summary>Expected</summary>

No. The disk does not depend on when the node comes back.

</details>

3. Crash a node forever in step 4 (remove its restart, add `sim.Crash`). Why does the run now last ten minutes?
<details><summary>Expected</summary>

The others keep telling a peer that never answers. With a node down for good nothing empties the queue, so add `sim.StopAt` to end the run.

</details>
