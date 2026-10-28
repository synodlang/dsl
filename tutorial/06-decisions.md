# 6. Decisions

## Goal

Make the game stop by itself after ten rounds.

## New in this chapter

<!-- new -->

## Steps

The game of chapter 5 never ends on its own; we stopped it from outside with `sim.StopAt`. The players should decide for themselves. That takes a **condition**: a value that is true or false, and a block that runs only when it is true.

`if` takes a condition and a block. `else` gives the block for the other case; it sits on its own line, at the same indentation as the `if`, right after its block. A condition with more than one word is in parentheses, like every argument. The simplest condition is a comparison: `less than` compares two numbers and gives a `bool`, a value of type `bool`, either `true` or `false`. A `bool` can also be kept: `Done` is a node field that starts as `false`.

In step 1 the node that gets a `Pong` checks the round. Below ten it pings again; at ten it has had enough and sets `Done` to `true`. Without `sim.StopAt` the run ends when nothing is left to happen, so the demo no longer needs it:

<!-- step 1 program=pingpong -->

The game ends after 2 seconds, with ten rounds counted by both nodes. But only the first node knows it is done. Step 2 teaches the other one the same, and makes the first node careful before serving. The comparison words are:

```
a is b           a is not b
a less than b    a more than b
a at least b     a at most b
```

`is` and `is not` compare any two values of the same kind: numbers, texts, nodes, bools. The other four compare numbers (or durations). Conditions combine with `and`, `or` and `not`, which work on bools:

```
(n.Peer is not sim.Nobody) and (not n.Done)
```

Each side of `and` is one argument, so each comparison has its own parentheses; and there is no precedence to remember. In step 2 the serving node starts only if it has a partner, the `Start` names a real round (`more than 0`) and it is not done, and a node that gets a `Ping` of round ten, or more, also marks itself done:

<!-- step 2 program=pingpong -->

In step 3 the demo checks the result after five seconds and reports it with captions. The first `if` uses `or` to join two comparisons: the game never started if either node played no round. The second compares two bools with a plain `is`: if the nodes are not both done, either neither is or only one is. The `else` block holds another `if`, because `else if` does not exist: you nest.

<!-- step 3 program=pingpong -->

Notice that the demo can read `a.pingpong.Done` and `a.pingpong.Rounds`: the demo sees the state of any node (it is the director of the show), nodes do not.

## What to watch

The messages go back and forth ten times and then the animation stops with nothing in flight. The final state shows `Done true` on both cards, and after five seconds the caption says so. In step 1 the second card still says `false`: only the one that heard the last answer is done.

## Why it looks like this

You may want to write "is not equal" as `not` around an equality. It parses, but it is not canonical:

<!-- why negated-is -->

`synod check` says `error[non-canonical]` and the hint shows the one spelling: `(n.Peer is not sim.Nobody)`. Synod has one way to say each thing. `synod fmt` rewrites `not (a is b)` into `a is not b`, and `not (a less than b)` into `a at least b`, so you never need to do it by hand. `not` stays for bools, like `not n.Done`.

## Exercises

1. Change the limit from 10 to 3 in both places. What does the final state show?
<details><summary>Expected</summary>

Three rounds: both nodes count 3 and both say `Done true`. (In step 1 only the first.)

</details>

2. Replace `at most 0` in the demo by `less than 1`. Is anything different?
<details><summary>Expected</summary>

Nothing: for whole numbers `at most 0` and `less than 1` mean the same. Both are canonical, so pick the one that reads best.

</details>

3. Use `more than` to say the same as `at least 10` in the `Ping` handler.
<details><summary>Expected</summary>

`Ping.round more than 9` is the same for whole rounds. `at least` reads better here.

</details>
