# Write a look

A look decides how a run is drawn: where the nodes sit, what a crash looks like,
which moment gets a pulse. It observes the trace and can never change what
happens ([the visual layer](../../spec/09-visual-layer.md)). If you write
no look, every program is drawn by `std/look/basic`. This guide makes a small
look for the module of [chapter 4](../04-first-node.md); [chapter 12](../12-looks.md)
teaches the same ideas step by step.

## 1. A package for the look

A look is an ordinary package. Add a folder `ticklook` next to the `tick`
package, with a file `ticklook/ticklook.synod`. The package that counts is the
one `synod new tick --module example.com/tick` writes.

```synod
# package ticklook
use std/look
use std/sim
use std/theme

# Every node gets the standard card, in a row instead of a ring
when sim.Started
  look.Layout "row"

when (sim.Added n)
  draw (look.Card for n) as card for n

# A crashed node fades and gets a red outline
when (sim.Crashed n)
  look.Fade (card of n) to 0.3
  look.Stroke (card of n) with theme.Critical
```

- `use std/look` is what makes the program bring its own look. From then on
  `std/look/basic` no longer runs, so the cards are yours to draw. You can
  keep the default and add to it with `use std/look/basic`.
- `when sim.Started` and `when (sim.Added n)` are observers. They run on the
  presentation clock, once for each such fact in the trace, and see the state
  as it was after that fact.
- `draw (look.Card for n) as card for n` draws the standard node card and
  attaches it to the node under the name `card`. Any observer can then write
  `card of n`.
- `look.Fade` and `look.Stroke` change the card; the change morphs rather than
  jumps.

## 2. Turn it on

A look is active when the program imports it. In the main file, import the
look package:

```
# package main
use example.com/tick/tick
use example.com/tick/ticklook
use std/sim

define main
  set first to (sim.Spawn tick)
  after 3s
    sim.Crash first
  sim.StopAt 6s
```

The import is used because the package contains observers. Run the folder
and watch: in the [Mac app](mac-app.md) the stage updates as you save, and on the
command line `synod run tick --summary` shows the same run as text. A
mistake in a look is a look failure: the simulation is still correct, the scene
stops at that fact and `synod run` says `look failed at` with the position.

## 3. Colors

Use the names of `std/theme` (`theme.Ink`, `theme.Paper`, `theme.Line`,
`theme.Quiet`, `theme.Accent`, `theme.Good`, `theme.Warning`,
`theme.Critical`) so your look matches the rest. To add your own, give a color
a name with a phrase that has no holes; a color literal is allowed only as its
result. The scene lists it as `ticklook.Hot`.

```synod
# package ticklook
# The color of a node that has counted a lot
define Hot
  result #E86F5A
```

The values of the standard palette are in the
[std/theme table](../../std/README.md) and the rules in
[§9.5](../../spec/09-visual-layer.md). Labels are drawn in the player's sans at
one size; a look cannot change it in Synod 0.2.

## 4. What you can draw

`std/look` has shapes (`Circle`, `Rect`, `Label`, `Line`, `Group`, `Card`),
changes (`Fill`, `Stroke`, `Fade`, `Pulse`, `Effect`), placement (`Layout`,
`Place`, `Orbit`, `Center`) and motion (`Tween`, `Travel`, `Flight`). The
[std/look reference](../../std/README.md) lists every phrase with its pattern and
the [layout](../../spec/09-visual-layer.md) rules are in §9.4. Two forms matter
most:

- `draw ... as name for key` attaches a shape to a node, or to a text or
  number key, for the whole run. `name of key` finds it again.
- An indented block after `draw` is reactive: it runs again whenever a field
  it reads changes, and the shape morphs from the old look to the new one.
  This is how a card shows `Count` changing without any animation code.

Looks and invariants are also how a program tells its story: the program emits
a fact and the look draws it ([chapter 13](../13-story.md)).

## Next

- [Publish a package](publish-a-package.md) puts the program and its look in
  a module others can require.
