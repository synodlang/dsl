# Claude integration (in `mac-app`)

The app drives the Claude Code CLI on the user's machine. The language
never depends on it; any agent that can follow `agent-skills` works the
same way.

## Invocation

```
claude -p "<request>" \
  --append-system-prompt "$(cat agent-skills/skills/synod/SKILL.md)" \
  --allowedTools "Read,Write,Edit,Bash(synod check:*),Bash(synod run:*),Bash(synod doc:*)" \
  --output-format stream-json
```

run with the workspace as the working directory. The skill tells Claude
how to look up phrases (`synod doc`), write canonical code, and verify
with `synod check` before finishing. The app streams progress lines into
the prompt bar.

## Flow for "animate X"

1. **Reuse first.** The skill tells Claude to search the workspace,
   `examples` and `synod doc` for existing packages. If a package already
   models X, Claude only writes a main file (`define main`, typically 10–30 lines).
2. **New behaviour only when missing.** Otherwise it writes a new
   package plus a main file.
3. **Verify.** Claude runs `synod check` and fixes diagnostics. Errors are
   short by design (`spec/11`), so each fix costs few tokens.
4. **Run.** Claude runs `synod run` and reads the last lines of the trace
   summary to confirm the scenario shows what was asked (for example,
   that a leader was actually elected), then stops.
5. The app re-runs and plays the result.

## Edits

"Slow down the crash" or "add a second proposer" are sent with the
current file paths; Claude edits in place with the `Edit` tool, so only
changed lines are generated.

## Captions

"Explain this run" sends `synod run --summary`, a compact trace summary
(facts grouped per step, under 2,000 tokens for typical runs), and asks
for `lesson.Note` lines to insert into `define main`. Facts come from the
simulator, so captions cannot invent behaviour.

## Budget rules

- Never paste whole traces or scenes into a prompt.
- Never ask the model for coordinates, colors or timings; looks decide those.
- One CLI session per request; reuse it for follow-up edits in the same window.
