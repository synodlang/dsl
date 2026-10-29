# 14. Packages

## Goal

Build a service on top of the rumor package, and use somebody else's package next to your own.

## New in this chapter

<!-- new -->

## Steps

Until now each package stood alone. Real systems are built in layers: a service uses a gossip library, an application uses the service. In Synod a package uses another one with `use`, which you know, and **running is transitive**: a node of a package also runs every package that package uses. A `service.Node` is also a `rumor.Node`.

Step 1 adds a `service` package that uses `rumor`. The service answers a question with what its rumor package knows. A client node is sent an `Ask` that names a server; it sends a `Question`; the server answers. The line that matters is `n.rumor.Known`: from outside, a field is reached through the package that declares it, `node.package.Field`, and it works because the node runs `rumor`. The demo builds a cluster of service nodes and hands the list to `rumor.Connect`, which wants `list of rumor.Node`: a `service.Node` is accepted wherever a `rumor.Node` is expected, because running `service` runs `rumor`:

<!-- step 1 program=service -->

The invariants of chapter 11 sit in `rumor` and make it harder to reuse. A rule belongs to whoever wants it, so step 2 moves them to a package of their own, `checks`. It declares no node, no start and no message: only phrases and `always`. A package with an invariant is active as soon as a program imports it, even if the program never names it, which is why the demos import `checks` and nothing else from it. The rules read `n.rumor.Known` because they look at `rumor` nodes:

<!-- step 2 program=service -->

The next package is not yours. Packages come from Git repositories, and a module says which ones it needs in `synod.mod`. We want the `gossip` package of the examples repository, tagged `v0.3.0`. Step 3 adds the line

```
require github.com/synodlang/examples v0.3.0
```

and `synod get` downloads it into `vendor/` (git ignores that folder) and writes a file `synod.sum` that pins the exact commit, so the program replays the same years from now. Commit `synod.sum`, never `vendor/`. Everything works offline once fetched.

The same works for looks and themes: the module `github.com/synodlang/looks` holds ready-made ones, fetched and imported like any package.

We also write our own package called `gossip`, and it builds on theirs. Both are named `gossip`, so the import of the other would be ambiguous inside ours. That is the one case for `use … as`: the import is given another name, `base`. A node of our `gossip` also runs `base`, so a `gossip.Node` is accepted by `base.Join`:

<!-- step 3 program=gossip -->

Step 4 reads what the base package knows, through our own node: `n.base.Known` and `n.base.Down` are exported fields of the package we imported. The phrase `Alive at` gives the number of nodes a node believes are alive, and the demo crashes one node and asks for the count at the end:

<!-- step 4 program=gossip -->

Notice what we did not write: a single line of the gossip algorithm. The package is somebody else's, versioned and pinned, and it still animates like your own.

## What to watch

Step 1: the answer travels from the server to the client, and the client's card gets the text. Step 2: the same picture, with the rules active. Steps 3 and 4: the nodes exchange heartbeats; after the crash, the others learn that `n3` is down, one after another at their own times, and the caption at the end says four of five are alive.

## Why it looks like this

`use … as` is for the collision of two last segments, and for nothing else. Renaming an import without a collision is refused, because it would give one package two names:

<!-- why unneeded-alias -->

`synod check` says `error[unneeded-alias]` and hints at the plain `use`.

## Exercises

1. In step 1 deliver the `Ask` to `item 0 of nodes` with `Of (item 0 of nodes)`, so that `n1` asks itself. What does `Heard` hold?
<details><summary>Expected</summary>

The same text: a node can ask itself, because sending to yourself goes through the network like any other message.

</details>

2. In step 4 crash two nodes. What does the caption say?
<details><summary>Expected</summary>

The count falls to 3 of 5 once both are noticed as down.

</details>

3. Remove `as base` and rename the uses of `base` to `gossip`. What does `synod check` say?
<details><summary>Expected</summary>

`error[duplicate-name]`: the qualifier `gossip` is already the name of this package, and the hint suggests `as <name>`.

</details>
