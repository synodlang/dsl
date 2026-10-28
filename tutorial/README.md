# Learn Synod

A bottom-up tutorial: it starts with the smallest program that shows
something and grows one idea at a time. Each chapter has a goal the current
program cannot reach yet, introduces exactly the syntax that closes the gap,
and starts from the code the previous chapter ended with. The format of a
chapter is in [`implementation/11-documentation.md`](../implementation/11-documentation.md);
[`coverage.md`](coverage.md) says which chapter teaches which form, type,
context and std package.

You need the `synod` command ([install](guides/install.md)). Every step is a
complete program: type it, run it with `synod run`, watch, then change it.

## Part I: One program, no nodes

1. [Hello](01-hello.md): show a sentence on the stage
2. [Values](02-values.md): show a computed sentence
3. [Time](03-time.md): show sentences at different moments, and ask the viewer

## Part II: Nodes and messages

4. [First node](04-first-node.md): put a node on the stage that counts
5. [Messages](05-messages.md): two nodes play ping-pong
6. [Decisions](06-decisions.md): stop after ten rounds
7. [Phrases](07-phrases.md): remove the repeated lines
8. [Many nodes](08-many-nodes.md): broadcast to every peer and count replies
9. [Chance](09-chance.md): spread a rumour to random peers

## Part III: When things go wrong

10. [Faults](10-faults.md): keep the rumour spreading through crashes and loss
11. [Invariants](11-invariants.md): say what must never happen, then break it

## Part IV: Seeing it

12. [Looks](12-looks.md): draw the system your way
13. [Story](13-story.md): make the important moment stand out

## Part V: Building real systems

14. [Packages](14-packages.md): build a service on top of the rumour package
15. [Raft](15-raft.md): build `examples/raft` from an empty folder

## Guides

- [Install the CLI](guides/install.md)
- [Use the Mac app](guides/mac-app.md)
- [Write a look](guides/write-a-look.md)
- [Publish a package](guides/publish-a-package.md)
- [Use Synod with Claude or another agent](guides/agents.md)

After chapter 15, go on with the [examples](https://github.com/synodlang/examples)
and the [specification](../spec/00-introduction.md).
