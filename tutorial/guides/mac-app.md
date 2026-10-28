# Use the Mac app

The Synod Mac app is an editor and a player in one window. It runs your program
with its own copy of `synod`, so you do not need the command line to use it.
It needs macOS 15 or later and works offline. The app bundles the compiler, the
player, the examples and the agent skills.

## Install

Download `Synod-<tag>-macos.zip` and `SHA256SUMS` from the
[releases](https://github.com/synodlang/mac-app/releases) page, then verify and
unpack:

```sh
shasum -a 256 -c SHA256SUMS
unzip Synod-*-macos.zip && mv Synod.app /Applications/
```

The app is ad-hoc signed but not notarized. After the first blocked launch,
open System Settings, Privacy & Security and choose "Open Anyway", or remove
the quarantine flag:

```sh
xattr -dr com.apple.quarantine /Applications/Synod.app
```

## Open a workspace

A workspace is a folder with a `synod.mod`, the module of
[chapter 4](../04-first-node.md) onwards.

- **File ▸ New Workspace…** (or the button on the welcome screen) makes a new
  module for you. The same screen has Open Examples, which opens the bundled
  examples, and a list of recent workspaces.
- **File ▸ Open Workspace…** (⌘O) opens an existing folder.
- **File ▸ Copy Examples to Folder…** copies the bundled examples to a folder you
  can edit, since the bundled copy is read-only.

The sidebar lists the workspace's programs, packages and the examples. Pick a
program to make it the one that runs.

## Run and watch

Press ⌘R, or just edit: the program runs again 300 ms after you stop typing.
The stage plays the scene. Space plays and pauses, the left and right arrows
step to the previous and next fact, and you can drag the timeline. The speed
goes from 0.25x to 4x, and a toggle chooses ordered or proportional mode (see
[the visual layer](../../spec/09-visual-layer.md), §9.6). Markers on the
timeline show questions, captions and broken invariants. A red banner appears
when an invariant breaks or the run fails.

The output goes to the app's cache and never into your workspace. Click a node
or a message on the stage to select it; the inspector shows the node's state at
that moment, the fields that just changed and the trace around it. Problems from
`synod check` and failed runs are listed too.

## Edit

The editor colors the source from `synod check --tokens` and underlines
problems as you type. ⌘. offers a quick fix, ⌘S formats and saves, and ⌘↓ goes
to the definition of the phrase at the caret. Type a few letters of a phrase,
or press ⌃Space, to pick it from a list of the phrases in scope. Tab moves
through the holes of the phrase you inserted.

## Predict

When a program asks with `lesson.Ask` (see [chapter 3](../03-time.md)),
playback stops at the question. Type your answer, or press 1 to 6 when the
question offers choices, then continue: the stage shows what really happened.
Answers are kept for each program.

## Break it

Break it changes the run without you writing code.

- Click a node to crash or restart it.
- Click a message to drop it.
- Drag a box around some nodes to partition them.

Each action adds a block to `define main` at the current simulated time,
formats the file and runs it again, so what you did is ordinary Synod you can
read, keep or edit. Undo Break it puts the file back. Invariants that break
are highlighted where they broke (see [chapter 11](../11-invariants.md)).

## Compare

Compare puts two runs side by side on one timeline: two seeds of the same
program (Next seed steps through them), or the program before and after a Break
it (With before Break it). Exit Compare returns to one stage.

## Export

**Run ▸ Export Video…** (⇧⌘E) opens the export sheet. Choose H.264 or HEVC,
1080p or 4K, 30 or 60 fps, or an animated GIF (480 to 960 px wide, 10 to 50
fps, with a time range).

## Ask Claude

The prompt bar (⌘K) asks your own Claude Code to write or change Synod source
in the workspace. It is off until you turn it on in Settings ▸ Claude, and it
needs the `claude` command installed and signed in with your own account. Edits
arrive in your open buffers as "Claude Edit" undo steps, a banner lists the
changed files, and Revert restores them. The app stores no key and opens no
network port. [Use Synod with Claude or another agent](agents.md) explains what
Claude can do and what stays your step.

## Next

- [Write a look](write-a-look.md) to change how a run is drawn.
- [Publish a package](publish-a-package.md) to share a module.
