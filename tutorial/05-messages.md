# 5. Messages

## Goal

Make two nodes play ping-pong over the network.

## New in this chapter

<!-- new -->

## Steps

Nodes share nothing. The only way one node learns something from another is a **message** sent over the network. We add a second package to the module, `pingpong/`, next to `counter/`. It holds two files again: the node's behaviour in `pingpong.synod`, and `demo.synod` with `define main`.

A message is a **record**: a named bundle of fields. A record type is declared with `has`, like a node, one `set field to value` line per field, with the value the field has when you do not give one. Our messages are `Ping` and `Pong`, each carrying the number of the round. `Start` is a third record, the signal that tells the first node to serve; its capital `Round` makes it readable from the demo.

`Node has` keeps a `Peer`, the node to talk to, which starts as `sim.Nobody`, the value that means "no node". A node is a value of type `node`; it has no fields of its own, it is just a name you can send to.

The behaviour is three `when` blocks. `when [n: Node] (…)` runs on every node of the package whenever the event in parentheses happens, with `n` bound to that node:

- `sim.Delivered Start` happens when the demo hands a `Start` to a node with `deliver`. The node answers by sending a `Ping`.
- `net.Received Ping from sender` happens when a `Ping` arrives. The words `Ping` and `sender` are new names for the block: `Ping` is the record that arrived, so `Ping.round` is its field, and `sender` is the node that sent it.
- `net.Received Pong from sender` is the same for a `Pong`.

`message Ping with round Start.Round` builds a record with `round` set. `net.Send … from n to p` sends it; the network takes some time to carry it. The `from n` repeats who is sending, because a node can only send as itself.

In the demo `a.pingpong.Peer` is a field of the node `a` that belongs to the package `pingpong`: from outside, a field is written `node.package.Field`. And `deliver (message pingpong.Start with Round 1) to a` builds a `Start` from another package, so the record is written `pingpong.Start`. Step 1 plays a single round:

<!-- step 1 program=pingpong -->

Run `synod run pingpong`. The summary shows one `Ping` going from `n1` to `n2`, and one `Pong` back.

To keep playing, the node that gets a `Pong` sends the next `Ping`, one round later, and the node that gets a `Ping` counts it too. Messages now need time to be seen: `net.Latency between 50ms and 150ms` makes every message take a random time between the two durations. Because nothing else stops the game, `sim.StopAt 3s` ends the run:

<!-- step 2 program=pingpong -->

## What to watch

Two cards, and a dot travelling between them for each message. The delays differ, so the rhythm is uneven, but the order is always `Ping`, `Pong`, `Ping`, … The `Rounds` counter of both nodes grows together. Run it again: the picture is identical, because the random delays come from a fixed seed. Chapter 9 explains how.

## Exercises

1. Make `n2` start the game instead of `n1`.
<details><summary>Expected</summary>

Give `b` the peer and deliver the `Start` to `b`: `set b.pingpong.Peer to a` and `deliver … to b`. The roles swap.

</details>

2. Change `net.Latency` to `500ms` and `1s`. What happens in the three seconds?
<details><summary>Expected</summary>

Only a couple of rounds fit before `sim.StopAt 3s` (two in the default run), since each message takes between half a second and a second.

</details>

3. Crash `b` after one second: add `after 1s` with `sim.Crash b` inside, before `sim.StopAt 3s`. What does `a` do?
<details><summary>Expected</summary>

`a` sends its next `Ping`, but a message that arrives at a crashed node is dropped (`net.Dropped`, reason `crashed`), no `Pong` comes back, and the game ends there.

</details>
