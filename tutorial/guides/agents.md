# Use Synod with Claude or another agent

An agent can write Synod well because the compiler checks every line: the
code is canonical, errors name the fix, and a run is deterministic. This guide
sets up Claude Code, other agents and the Mac app so an agent can do the
work and you can watch it. You need the `synod` command
([Install the CLI](install.md)).

## Claude Code: the plugin

The `agent-skills` plugin teaches Claude Code to look for an existing package,
write Synod, check it and confirm the run shows what you asked.

```sh
claude plugin marketplace add synodlang/agent-skills
claude plugin install synod@synodlang
```

Inside a Claude Code session the same two steps are
`/plugin marketplace add synodlang/agent-skills` and
`/plugin install synod@synodlang`. The `synod` skill then loads when a task
mentions Synod, `.synod` files or `synod.mod`. These commands follow the Claude
Code documentation and the plugin's README, which says it has not yet run them
against a fresh install; if one fails, the README of
[agent-skills](https://github.com/synodlang/agent-skills) is the reference.

Then ask in plain words, for example "write a Synod program that shows three
nodes electing a leader, then crash the leader".

## Other agents: AGENTS.md

Any agent that reads an `AGENTS.md` can use the same instructions. Copy
`AGENTS.md` and the folder `skills/synod/reference/` from the
[agent-skills](https://github.com/synodlang/agent-skills) repository into your
project, keeping the folder next to the file. `AGENTS.md` is the skill's entry
point; the reference folder holds the grammar, the phrase rules, the std
catalog, the error list and a copy of the examples.

## The tools over MCP: `synod mcp`

An agent without a shell, or any Claude Code session, can use the compiler as
MCP tools: `doc`, `explain`, `check`, `run_summary` and `fmt`. They run on the
local `synod` and do not touch the network.

```sh
claude mcp add synod -- synod mcp
```

Add `--root <dir>` to name the module the tools work in.

## If an agent has a shell

The tutorial's commands are the ones an agent should use:

| Command | Why an agent runs it |
| --- | --- |
| `synod doc std/net` or `synod doc` | find phrases and packages that exist, with their patterns |
| `synod fmt` | the compiler accepts only canonical code, so format after every edit |
| `synod check` | one line per diagnostic when piped, with a hint, the same on every run |
| `synod check --json` | exact spans and fixes (`edits`) when it wants to apply a fix |
| `synod explain <code>` | an example and the fix for a diagnostic code |
| `synod run <folder> --summary` | the run as compact text: steps, invariants, final state |

An agent should never paste a whole trace into a prompt and never invent
coordinates, colors or timings: looks decide those ([Write a look](write-a-look.md)).
`synod get` stays your step unless you tell the agent to run it, because it
downloads code ([Publish a package](publish-a-package.md)).

## The Mac app: prompt bar and editor tools

The [Mac app](mac-app.md) runs your own Claude Code for you.

1. Install Claude Code and sign in with your own account.
2. Open Settings ▸ Claude and enable the prompt bar. If `claude` is not found,
   set its path there.
3. Press ⌘K in a workspace and write the request, for example "animate Raft
   with 5 nodes and a leader crash".

The app starts `claude` with the skill from agent-skills and a short list of
allowed tools: `Read`, `Glob` and `Grep`, the editor tools, and the `synod mcp`
tools `doc`, `explain`, `check` and `run_summary`. There is no shell, no
subagent and no direct file writes. You see live progress and can Stop. The
edits land in your open buffers as "Claude Edit" undo steps, and a banner lists
the changed files. Revert restores every file to how it was before the request.
The app stores no key, opens no network port and never logs the text of your
files or prompts.

### From your own Terminal

If you prefer to drive Claude in Terminal, the app can share its editor with
it. In Settings ▸ Claude, turn on external Terminal access, choose Copy MCP
config and run the `claude mcp add synod-editor ...` line it copied. A `claude`
started in Terminal then reads and edits the buffers open in the app, with the
same "Claude Edit" undo steps. The setting is off at every launch, and works
until you click Disconnect or quit the app.

## Check the work yourself

Whoever wrote the program, trust the compiler and your own eyes:

```sh
synod fmt --check
synod check
synod run <folder> --summary
```

Then play the scene in the app and use Predict, Break it and Compare to see
whether the system really does what the prompt said.
