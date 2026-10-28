# 15. Raft

## Goal

Build the `raft` example from an empty file, one piece at a time.

## New in this chapter

<!-- new -->

## Steps

Nothing in this chapter is new. You know every form it needs; the work is to put them together in the right order. Raft is an algorithm by which a group of nodes agrees on one **leader**, even when nodes crash. The idea fits in one page:

- A node is a **follower**, a **candidate** or a **leader**, and has a **term**, a counter that only grows.
- A follower that hears nothing for a while becomes a candidate: it adds one to its term and asks the others for their vote.
- A node votes for at most one candidate per term. A candidate with a majority of votes becomes the leader.
- The leader sends heartbeats, so that nobody starts another election.
- Seeing a higher term, anyone steps down.

We build it in seven steps, each adding one piece. Step 1 has the node: its role, its term, who it voted for, its peers, and the phrase that makes it a follower. Names that start in lower case, such as `votes` and `lastHeard`, are private to the package. The demo builds a cluster; nothing happens yet:

<!-- step 1 program=raft -->

Step 2 starts elections. Waiting is done with `after` and a random duration, so that the nodes do not all become candidates at the same instant and split the vote again and again. A node watches the clock: if it is not a leader and has heard nothing for 150ms, it starts an election and sends a `RequestVote` to every peer. Nobody answers yet, so every node keeps trying:

<!-- step 2 program=raft -->

Step 3 counts votes. A node grants its vote when the candidate's term is its own and it has not voted for anyone else. It answers with a `Vote` that says whether it was granted, and a candidate that collects a majority of the cluster (`Majority of` is half and one) becomes the leader. Any message with a higher term makes the receiver step down first. There are leaders now, but they say nothing, so the followers time out and start new elections:

<!-- step 3 program=raft -->

Step 4 adds the heartbeat. The leader sends one every 50ms, and a follower that hears one remembers when, so its election timer never runs out:

<!-- step 4 program=raft -->

Now the rule that must always hold. Step 5 adds the invariant of chapter 11: never two leaders in the same term. It also makes the system announce a new leader with an event, `Elected`, for the look to catch, and a phrase `Leaders` that lists the running leaders. This is the whole `raft.synod` of the examples, byte for byte:

<!-- step 5 program=raft -->

Step 6 adds a look in a package of its own, `raftlook`, and imports it from the demo. It replaces the card of each node with one whose color follows its role, and pulses and glows the card of a new leader:

<!-- step 6 program=raft -->

The last step is the demo of the example. After two seconds it crashes the leader, asks the viewer to predict what the followers do, and restarts the old leader three seconds later, when it comes back as a follower:

<!-- step 7 program=raft -->

Compare it with the file `raft/raft.synod` in the [examples](https://github.com/synodlang/examples) repository: only the module path differs.

## What to watch

At the start, five followers, then a candidate, then a leader in amber that sends dots to every other card. After the crash, the dots stop; one follower times out first, becomes a candidate (yellow), and wins a new term. The old leader returns as a follower: it sees a higher term and steps down.

## Exercises

1. Change `random.TimeBetween 150ms and 300ms` to `random.TimeBetween 150ms and 160ms`. What happens to the elections?
<details><summary>Expected</summary>

Candidates collide more often, so more terms pass before somebody wins a majority. A safe system does not depend on luck, but a fast one does.

</details>

2. Make the cluster 3 nodes, and crash two.
<details><summary>Expected</summary>

No leader can be elected again: a majority of three is two, and only one node is left. The system stays safe, but cannot make progress.

</details>

3. Turn `LeadersAgree` into a rule that is false by design: replace `n.Term is in terms` with `n.Term is not in terms`.
<details><summary>Expected</summary>

`sim.Broken` fires as soon as there is a leader, which shows that the rule is checked after every step.

</details>
