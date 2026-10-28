# 3. Time

## Goal

Show sentences at different moments, and ask the viewer a question.

## New in this chapter

<!-- new -->

## Steps

So far everything happened at the first instant. A Synod program runs on a simulated clock, and `after 2s` runs its block when that clock reaches two seconds. `2s` is a duration, a value of type `time`; `ms` and `s` are the two units. We take the program of chapter 2 and move its last caption later:

<!-- step 1 program=hello -->

`time.Now` is the current simulated time. Put it in the caption to see the clock:

<!-- step 2 program=hello -->

A caption can also ask. `lesson.Ask` pauses the show and asks the viewer to predict what comes next, before it happens:

<!-- step 3 program=hello -->

Give the viewer choices with `choosing` and a list of two to six texts:

<!-- step 4 program=hello -->

Last, `sim.StopAt` sets the moment the run ends at the latest. This run has nothing left to do after four seconds, so it ends then; if you set `sim.StopAt 3s`, the question never appears:

<!-- step 5 program=hello -->

## What to watch

Captions arrive at 0s, 2s and 4s. At 4s the player stops and shows the question with its two buttons. The clock is simulated: the run does not wait two real seconds, and running it again gives exactly the same events. This is what later lets you replay a failure.

## Exercises

1. Change `after 2s` to `after 1500ms`. What does the caption say?
<details><summary>Expected</summary>

It appears at 1500ms and reads `At 1500ms each pays 5.5`.

</details>

2. Set `sim.StopAt 3s`.
<details><summary>Expected</summary>

The run ends at 3s, so the question at 4s never appears.

</details>

3. Remove `choosing {"Ana", "Ben"}`.
<details><summary>Expected</summary>

The question is shown without buttons; the viewer just thinks of an answer.

</details>
