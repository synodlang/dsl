# 9. Chance

## Goal

Spread a rumor through six nodes, each telling a random peer at a random moment.

## New in this chapter

<!-- new -->

## Steps

The game of the last chapters was a conversation with a fixed rhythm. Real systems spread news differently: a node knows something and tells somebody, then somebody else, a little later. Nobody coordinates. This is a **rumor**, and it needs randomness: who to tell, and when.

Synod has randomness, and it has no surprises. The simulator holds one random number generator that starts from a **seed**, a number, so the same program with the same seed gives exactly the same run, every time. `std/random` offers the phrases:

- `random.Pick l` is an item of the list `l`, every item equally likely.
- `random.TimeBetween a and b` is a random duration between `a` and `b`.
- `random.Chance p` is `true` with probability `p`, between 0 and 1.

(`random.Between` does the same for numbers.)

We start a new package, `rumor/`, next to the others. It will be the one we grow for the next chapters, so it is built to be extended. A `Rumor` is a record with a `Text`. A node's `Known` is the rumor it knows, or `""` when it knows none, and `Peers` is who it can talk to.

`Learn` is the heart: a node that does not know a rumor believes it, and starts telling it. `Tell` sends the rumor to `random.Pick n.Peers`, then schedules itself again after a random pause, `after (random.TimeBetween 200ms and 600ms)`; the pause is the one item of `after`, so it is in parentheses. Two `when` blocks call `Learn`: one for the rumor delivered by the demo, one for the rumor received from another node. The demo makes six nodes, connects them (the same `Connect` as in chapter 8) and gives the rumor to the first:

<!-- step 1 program=rumor -->

Run `synod run rumor`. Every node tells forever, so the run needs `sim.StopAt 5s`. That is not how rumors spread: people lose interest. In step 2 each telling is followed by another with probability 0.7 only, `random.Chance 0.7`, and the run ends by itself when everybody has gone quiet:

<!-- step 2 program=rumor -->

Every run of step 2 looks the same, because the seed is always 1 unless you say otherwise. Change it with a flag, `synod run rumor --seed 4`, or in the program with `sim.Seed 4`, which the flag overrides. Step 3 fixes the seed in the demo:

<!-- step 3 program=rumor -->

With seed 4 the rumor dies out before reaching `n4`, and its card stays empty. Another seed, another story, and either one repeats exactly when you run it again: that is how you will replay a bug.

## What to watch

Arrows leave a node at uneven moments and land on random peers. A card lights up the first time it hears the rumor. Count the messages in step 2: most are wasted on nodes that already know. Gossip is wasteful on purpose, because it is robust. With seed 4 one node is never reached.

## Exercises

1. Run step 2 with `--seed 2`, `3`, `5`. How many nodes know the rumor at the end?
<details><summary>Expected</summary>

Usually all six; seed 4 leaves five. Which one depends on the draws, but the same seed always gives the same answer.

</details>

2. Change `random.Chance 0.7` to `0.95`. What happens to the number of messages?
<details><summary>Expected</summary>

More messages, since nodes keep telling much longer; the rumor reaches everyone almost surely.

</details>

3. Change `random.Chance 0.7` to `0.3` with seed 4. Does the rumor reach everyone?
<details><summary>Expected</summary>

Less likely: nodes lose interest sooner. Try several seeds.

</details>
