# Claude integration (in `mac-app`)

The app drives the Claude Code CLI on the user's machine. The language
never depends on it; any agent that can follow `agent-skills` works the
same way. There is no in-app model client, no API key, no Keychain use and
no MCP server: the app never asks for, stores or passes a credential, and
Claude Code handles its own login (decision 96).

## Opt-in

The prompt bar is off until the user turns it on in Settings, Claude ("Enable
the prompt bar (uses Claude Code)"). The sheet that appears says what happens:
the app runs your `claude` command in the workspace; your request and the
files Claude reads go to Anthropic under your Claude Code account; the app
stores no key. With the setting off the app never launches `claude`.

## Invocation

```
<claude> -p "<request>" --output-format stream-json --verbose \
  --append-system-prompt "<SKILL.md body without front matter>

The reference files are in <skills>/skills/synod/reference/." \
  --add-dir <skills>/skills/synod \
  --allowedTools "Read,Glob,Grep,Write,Edit,Bash(synod fmt:*),Bash(synod check:*),Bash(synod run:*),Bash(synod doc:*),Bash(synod explain:*),Bash(synod get:*)" \
  --disallowedTools "WebFetch,WebSearch" \
  [--model <m>] [--resume <session_id>]
```

- The working directory is the module root. `<skills>` is the bundled
  `agent-skills`. `--verbose` is needed for `stream-json` with `-p`. `--model`
  is the Settings model (default `claude-opus-5-5`; empty adds no flag).
  `--resume` carries the session id of an earlier request in the same window.
- The skill tells Claude how to look up phrases (`synod doc`), write canonical
  code (`synod fmt`), verify with `synod check`, explain a diagnostic
  (`synod explain`) and fetch the example packages (`synod get`). The app
  streams progress lines into the prompt bar.
- The tools are the whole allowlist: Claude reads and writes files and runs
  only those `synod` commands (it writes Synod source, decision 15).
- The flags above are the documented ones of the Claude Code CLI; the
  implementer checks them against `claude --help` of the installed version and
  reports any difference.

### Environment

Only an allowlist is passed; nothing else, `ANTHROPIC_*` included.

| Variable | Value |
| --- | --- |
| `HOME`, `USER`, `LOGNAME`, `SHELL`, `LANG`, `LC_ALL`, `TMPDIR` | from the app's environment, when set |
| `PATH` | `<folder of the bundled synod>:<folder of claude>:/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin` |
| `NO_COLOR` | `1` |
| `TERM` | `dumb` |

## Flow for "animate X"

1. **Reuse first.** The skill tells Claude to search the workspace, run
   `synod doc` for the visible packages and, to use the `examples`, run
   `synod get github.com/synodlang/examples@v0.2.1` and import from
   `vendor/`. If a package already models X, Claude only writes a main file
   (`define main`, typically 10–30 lines), and never copies, rewrites or edits
   an existing package or `vendor/`.
2. **New behaviour only when missing.** Otherwise it writes a new
   package plus a main file.
3. **Verify.** Claude runs `synod fmt` and `synod check` and fixes
   diagnostics. Errors are short by design (`spec/11`), so each fix costs few
   tokens; `synod explain <code>` explains one.
4. **Run.** Claude runs `synod run --summary` and reads the last lines of the
   trace summary to confirm the scenario shows what was asked (for example,
   that a leader was actually elected), then stops.
5. The app reloads the changed files, selects and opens the newest added or
   changed main file's program, re-runs and plays the result, and offers
   Revert (`06-mac-app.md`).

## Edits

"Slow down the crash" or "add a second proposer" are sent with the
current file paths; Claude edits in place with the `Edit` tool, so only
changed lines are generated. A follow-up uses `--resume`.

## Captions

"Explain this run" sends this request (enabled when a run exists):

```
Explain this run: run "synod run <program> --summary", then add lesson.Note lines to define main in <main file> that caption what happens, at the times it happens. Do not change the behaviour.
```

`synod run --summary` is a compact trace summary (facts grouped per step,
under 2,000 tokens for typical runs). Facts come from the simulator, so
captions cannot invent behaviour.

## Budget rules

- Never paste whole traces or scenes into a prompt.
- Never ask the model for coordinates, colors or timings; looks decide those.
- One CLI session per request; reuse it for follow-up edits in the same window.
