# 4. First node

## Goal

Put a node on the stage that counts, then start a second one and crash the first.

## New in this chapter

<!-- new -->

## Steps

Until now one program did everything. A real system is many small programs running at once, each with its own memory. Synod calls such a program a **node**, and the code that describes how a node behaves lives in a **package**: a folder of `.synod` files.

Packages live in a **module**, a folder with a file `synod.mod` at its root. From here on you keep one module that grows from chapter to chapter. Create it with `synod new`:

```
synod new tutorial --module example.com/tutorial
cd tutorial
```

The command writes `synod.mod`, a `.gitignore` and a first package, `tutorial/`, with two files. Step 1 is exactly that output:

<!-- step 1 program=tutorial -->

Two files, two jobs. `tutorial/tutorial.synod` describes the node. `Node has` lists what each node remembers: here a number `Count`, which starts at 0. `define [n: Node] start` is what a node does when it is created. The hole `[n: Node]` gives the node a name, `n`, inside the block, and `n.Count` reads or writes a field of that node: a `.` joins a node to what belongs to it. `time.Every 1s` runs its block now and then once a second, and `set n.Count to (n.Count plus 1)` is the same `set` you know from chapter 2, now on a field instead of a local.

`tutorial/demo.synod` is the program. It starts with `# package main`, as in chapters 1 to 3, and uses the package it lives next to with `use example.com/tutorial/tutorial`: the module path, then the package folder. `sim.Spawn tutorial` creates one node that runs that package, and gives it back, so `set first to (…)` keeps it. Three seconds later `sim.Crash first` stops the node. Run it:

```
synod run tutorial
```

The package is named `tutorial`, like the module, which will get confusing. Rename the folder to `counter`, and change the header and the `use` lines to match. Step 2 is the same program with the new name:

<!-- step 2 program=counter -->

The header must always say the folder's name. Rename only the folder and `synod check` stops you, see the note at the end of the chapter.

One node is lonely. Step 3 spawns two nodes at the start of `define main` and crashes them at different moments. Each node has its own `Count`, so the two counters are independent, and a crashed node stays where it is:

<!-- step 3 program=counter -->

Last, a word about capitals. In Synod a capital letter has exactly one meaning: **exported**, visible from other packages. `Count` starts with one, so the demo, the animation and other packages can read it. Step 4 adds a lower-case field, `ticks`, that only this package can see:

<!-- step 4 program=counter -->

Run it again and compare the final lines: `ticks` is not among the fields shown, because nobody outside `counter` may look at it. Fields, phrases and records that other packages need are capitalized; everything else is not.

## What to watch

Each node is a card on the stage with its `Count`. The counter of the first node goes up once a second until the crash, when its card shows that the node is down; the second node keeps going. Look at the trace: the node starts at 0s with `Count` 1, because `time.Every` runs its block right away and then every second.

## Why it looks like this

The package header is the folder name, and the compiler checks it, so that a file can never say one thing while its place says another. In this module the folder `counter` still holds a file that says `# package tutorial`:

<!-- why bad-header -->

`synod check` answers `error[bad-header]`: the package named in the file must be the folder name (or `main`). Fix the header, not the folder.

## Exercises

1. Predict `Count` of both nodes at the end of step 3, then run it.
<details><summary>Expected</summary>

The first node ticks at 0s, 1s and 2s and is crashed at 3s before its fourth tick, so it ends at 3. The second ticks at 0s to 4s and ends at 5.

</details>

2. In step 3 swap the two crashed names, so that `second` is crashed at 3s and `first` at 5s. What changes?
<details><summary>Expected</summary>

The second node stops at 3s with `Count` 3, and the first keeps counting until the crash at 5s and ends at 5: the two final counts swap.

</details>

3. Make the counter go up by two each second.
<details><summary>Expected</summary>

Change `(n.Count plus 1)` to `(n.Count plus 2)`. Counts go 2, 4, 6…

</details>
