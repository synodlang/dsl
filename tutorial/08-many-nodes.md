# 8. Many nodes

## Goal

Ask every peer of a five-node cluster, count the answers, and move on when a majority has answered.

## New in this chapter

<!-- new -->

## Steps

Two nodes are a conversation; five are a system. A node now needs to remember many peers, and to count. For that Synod has **lists** (ordered values, `{a, b, c}`, which you met in chapter 2) and **maps** (a value for each key). The empty one is written `{}`: one spelling for an empty list and an empty map. Which of the two it is, and of what, the compiler works out from how you use it.

Step 1 turns the single `Peer` into a list `Peers`, starting as `{}`. The new phrase `Connect` has a hole `[ns: list of Node]`: the type of a list says what it holds, always, in a hole too. `each n in ns` runs its block once for each item, with `n` naming the item. `lists.Without n from ns` is a copy of the list without `n`: each node's peers are everybody else. The demo builds a cluster with `sim.Cluster of 5 running pingpong`, which spawns five nodes and gives back the list. `item 0 of nodes` is the first. `Start` makes the first node ping each of its peers with `each p in n.Peers`:

<!-- step 1 program=pingpong -->

Everyone now plays against the first. But nobody counts who answered. Step 2 changes the rule of the game: a round ends when a **majority** has answered, and then the next one starts.

The asking node keeps a map `Heard`, from node to the last round that node answered. Writing into a map is `set item sender of n.Heard to Pong.round`; the key is the sender, the value the round. `item k of m` would read it. `Answerers` is a value phrase with a hole of type `map of node to number`: it goes through the map with `each k, v in heard`, key and value, and keeps the keys whose value is the round asked about. `lists.Append k to who` adds one item. `sender is in n.Peers` checks membership.

`Majority of` shows that phrase heads can be followed by more words: `Majority of 5` is 3. It divides by two and rounds down with `math.Floor`, then adds one. `lists.Size` counts the items of a list. The asking node counts itself, which is why there are `plus 1`s:

<!-- step 2 program=pingpong -->

The run is deterministic: round by round the asking node moves on only when three of five have answered. Late answers to old rounds are ignored by `Pong.round is n.Round`.

Step 3 changes how the group is formed. Only the first node is told about the others, as two small groups joined with `lists.Extend near with far`. Everybody else learns who is asking, when a `Ping` arrives: `sender is not in n.Peers` is the opposite of `is in`. At the end of the demo the caption reads out what the first node heard. `maps.Size` is the number of keys; `maps.Keys` the keys in ascending order, a list:

<!-- step 3 program=pingpong -->

There is one type the table of types shows and you will not write: `value`, "any type". It appears only in the standard library's own patterns, such as `lists.Size [l: list of value]`, so that one phrase serves every kind of list.

## What to watch

The first card sends four messages to its neighbours, and the answers come back at different times. The round moves on after the third answer, before the slowest one has arrived. In the final state the first node says `Heard` with every peer's last round, and the others each know one peer, the first.

## Why it looks like this

An empty `{}` alone says nothing about its type. Here a local holds `{}` and then only goes through `text.From`, which takes any value:

<!-- why ambiguous-empty -->

`synod check` says `error[ambiguous-empty]` with a hint: use the empty value with `std/lists` or `std/maps`, or assign it where the type is known. A field like `Peers` is fine, because the program appends nodes to it somewhere.

## Exercises

1. Make the cluster 7 nodes in step 2. How many answers does a round need?
<details><summary>Expected</summary>

`Majority of 7` is 4: the asking node and three others. Rounds take about as long as the third-fastest answer.

</details>

2. In step 2, add `after 1ms` and crash two of the other nodes inside it, with `sim.Crash (item 1 of nodes)` and so on. Does the game finish?
<details><summary>Expected</summary>

Yes: three nodes still make a majority of five. Crash a third and the round never completes, and the game stops with `Done false`.

</details>

3. In step 3, print how many peers the third node knows at the end.
<details><summary>Expected</summary>

One: the first node, learnt from its `Ping`. Set `third` to `(item 0 of near)` and read `third.pingpong.Peers` in the caption, through `lists.Size`.

</details>
