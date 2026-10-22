# Claude integration (in `mac-app`)

The app drives the Claude Code CLI on the user's machine. The language
never depends on it; any agent that can follow `agent-skills` works the
same way. There is no in-app model client, no API key and no Keychain use:
the app never asks for, stores or passes a credential of the user, and Claude
Code handles its own login (decision 96). Claude changes the workspace through
two MCP servers (decisions 98 to 102): the editor server of the app, and
`synod mcp` of the compiler.

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

The reference files are in <skills>/skills/synod/reference/.

In this workspace, edit Synod files only with the synod-editor tools: read_buffer before editing, apply_edits or replace_buffer to change, create_file for new files, get_diagnostics (fresh) until clean, then save_files, and only then run_summary. Use the synod tools for doc, explain, check and run_summary." \
  --add-dir <skills>/skills/synod \
  --mcp-config <config file> --strict-mcp-config \
  --allowedTools "Read,Glob,Grep,mcp__synod-editor__*,mcp__synod__doc,mcp__synod__explain,mcp__synod__check,mcp__synod__run_summary,Bash(synod get:*)" \
  --disallowedTools "Write,Edit,MultiEdit,NotebookEdit,WebFetch,WebSearch,mcp__synod__fmt" \
  [--model <m>] [--resume <session_id>]
```

- The working directory is the module root. `<skills>` is the bundled
  `agent-skills`. `--verbose` is needed for `stream-json` with `-p`. `--model`
  is the Settings model (default `claude-opus-5-5`; empty adds no flag).
  `--resume` carries the session id of an earlier request in the same window.
- `<config file>` is `$TMPDIR/synod-mcp-<8 hex>/claude-<random>.json`, mode
  0600, written for the request and deleted when the process exits, whatever
  the reason:

  ```json
  {"mcpServers":{
    "synod-editor":{"type":"stdio","command":"<app>/Contents/Helpers/synod-editor-mcp","args":["--socket","<sock>"],"env":{"SYNOD_EDITOR_TOKEN":"<token>"}},
    "synod":{"type":"stdio","command":"<bundled synod>","args":["mcp"],"env":{}}}}
  ```

- The skill tells Claude how to look up phrases (`synod doc`), verify with
  the checker, explain a diagnostic (`synod explain`) and fetch the example
  packages (`synod get`). Files are formatted by Save, so there is no `fmt` tool
  in the request.
- The tools are the whole allowlist: Claude reads files, changes buffers
  through the editor server, uses the `synod` tools and runs only
  `synod get` in a shell (it writes Synod source, decision 15). The app streams
  progress lines (tool names included) into the prompt bar.
- The flags above are the documented ones of the Claude Code CLI; the
  implementer checks them, the tool names and the `type: "stdio"` config against
  `claude --help` of the installed version and reports any difference.

### Environment

Only an allowlist is passed; nothing else, `ANTHROPIC_*` included. The editor
token is in the config file, never in the environment of `claude` and never
in an argument.

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
3. **Verify.** Claude changes buffers with `apply_edits`, `replace_buffer` and
   `create_file`, reads `get_diagnostics` (fresh) and fixes diagnostics. Errors
   are short by design (`spec/11`), so each fix costs few tokens;
   `explain` explains one.
4. **Save, then run.** A run reads saved files (decision 81), so Claude calls
   `save_files` (the editor's Save: atomic write, then `synod fmt`) and only then
   `run_summary`, and reads the last lines of the summary to confirm the
   scenario shows what was asked (for example, that a leader was actually
   elected), then stops.
5. The app opens the newest added or changed main file's program, re-runs and
   plays the result, and offers Save All (when a buffer is unsaved) and
   Revert (`06-mac-app.md`).

## Edits

"Slow down the crash" or "add a second proposer" are sent with the
current file paths; Claude changes the open buffers with `apply_edits`, so only
changed lines are generated and every change is one undo group named "Claude
Edit". A follow-up uses `--resume`.

## Captions

"Explain this run" sends this request (enabled when a run exists):

```
Explain this run: run "synod run <program> --summary", then add lesson.Note lines to define main in <main file> that caption what happens, at the times it happens. Do not change the behaviour.
```

Claude reads the summary with the `run_summary` tool, which is the same
text as `synod run --summary`, a compact trace summary (facts grouped per step,
under 2,000 tokens for typical runs). Facts come from the simulator, so
captions cannot invent behaviour.

## Editor server (MCP)

The editor server lives in the Mac app and works on the buffers of the window
whose module root is the working folder. Its name is `synod-editor`; Claude Code
exposes its tools as `mcp__synod-editor__<tool>`.

**Transport.** Claude Code starts the bundled bridge
`<app>/Contents/Helpers/synod-editor-mcp --socket <path> [--workspace <module
root>]` as an ordinary stdio MCP server, with `SYNOD_EDITOR_TOKEN` in its
environment. The app listens on a Unix socket; nothing in the app listens on a
network, and a browser or another user's process cannot reach it (decision 98).

1. The bridge connects to the socket and sends one line:
   `{"synodEditorBridge":1,"token":"<64 hex>","workspace":"<abs path or null>"}`.
2. The app answers `{"ok":true}` or closes the connection.
3. After that the bridge copies stdin lines to the socket and socket lines to
   stdout, unchanged, until either side closes. Each line is one JSON-RPC
   message. The bridge never interprets MCP.
4. On a refused handshake or a missing socket the bridge exits with code 1 and
   the stderr line `synod-editor-mcp: Synod is not accepting editor connections`.

The socket is `$TMPDIR/synod-mcp-<8 hex>/editor.sock`: the folder is 0700, the
socket 0600, and the path stays under the 104-byte `sun_path` limit. There is
one listener per app process, created on demand and removed, with its folder,
when no session needs it. Tokens are 32 random bytes (`SecRandomCopyBytes`)
as hex, compared in constant time. A prompt-bar token is one per request, bound
to that window and revoked when the `claude` process exits. The external token
(below) is one per app run, kept in memory only. These tokens are the app's own
and short-lived; they are not a credential of the user. Without a prompt-bar
request or the external setting there is no listener and no socket file.

**Protocol** (both servers, decision 102). Stdio, UTF-8, one JSON-RPC 2.0
message per line, no embedded newlines, no batches. Methods: `initialize`
(answers `protocolVersion`, `capabilities: {"tools":{"listChanged":false}}` and
`serverInfo`), `notifications/initialized`, `ping`, `tools/list` (one page),
`tools/call` and `notifications/cancelled` (the editor server cancels a waiting
`get_diagnostics`). Everything else is error -32601. Errors: parse error
-32700, invalid request -32600, unknown tool or bad arguments -32602; a tool
failure is an `isError` result. The supported protocol versions are
`2025-11-25`, `2025-06-18` and `2025-03-26`; the server echoes the client's
when supported and otherwise answers `2025-11-25`. The implementer checks these
strings against modelcontextprotocol.io and what Claude Code sends, and
reports any difference.

**Tools** (decision 99). Every input schema is an object with
`additionalProperties: false`; results carry `structuredContent` and the same
JSON as one `text` item.

- `path` is a module-relative slash path, rejected as `outside-module` when it
  is absolute, contains `..` or resolves through a symbolic link out of the
  module. Writable files are `.synod` files and `synod.mod` outside `vendor/`,
  `out/` and dot or underscore folders; other text files of at most 1 MB are
  readable. Positions are 1-based lines and code-point columns as in
  `check.json`, ranges end exclusive. `version` is the buffer's change counter
  (`0` for a file that is not open, read from disk).
- Failures start with a code: `not-found`, `outside-module`, `read-only`,
  `version-mismatch`, `bad-range`, `exists`, `too-large`, `busy` or
  `bad-arguments`.

| Tool | Input | Output |
| --- | --- | --- |
| `list_files` | none | `{"files":[{"path","open","dirty","readOnly","version"}]}` in byte order of path |
| `read_buffer` | `path`, optional `startLine`, `endLine` (inclusive, clamped) | `{"path","version","dirty","readOnly","lineCount","startLine","endLine","text"}`, unsaved changes included |
| `get_diagnostics` | optional `path`, `fresh` (default true) | `{"stale","diagnostics":[{"path","line","col","endLine","endCol","severity","code","message","hint"}]}`; `fresh` runs the overlay check of decision 80 and waits up to 5 s, then returns the last result with `stale: true` |
| `get_selection` | none | `{"path","version","caret":{"line","col"},"selections":[{"line","col","endLine","endCol","text"}]}`, with nulls when nothing is open |
| `apply_edits` | `path`, `version`, `edits` (1 to 1000 of `line`, `col`, `endLine`, `endCol`, `text`) | `{"path","version","dirty":true}` |
| `replace_buffer` | `path`, `version`, `text` (at most 1 MB) | as `apply_edits` |
| `create_file` | `path` ending in `.synod`, `text` (at most 1 MB) | `{"path","version","formatted"}` |
| `save_files` | `paths` (at least one) | `{"saved":[{"path","version","formatted"}],"failed":[{"path","message"}]}` |
| `open_file` | `path`, optional `line`, `col` | `{"path"}` |

- `apply_edits` needs the buffer's current `version`, else `version-mismatch:
  buffer is at version N; read it again`. Edits must not overlap and must stay
  inside the text, else `bad-range`. They are applied in descending position
  order as one undo group named "Claude Edit". A file that is not open is opened
  first. The file stays unsaved.
- `replace_buffer` replaces the whole text as one undoable change; the file
  stays unsaved.
- `create_file` writes a new file atomically (missing folders are created),
  then runs `synod fmt` on it, as Save does, and opens it. An existing file is
  `exists`. A new file has to be on disk to be checked (decision 80).
- `save_files` is the editor's Save (decision 81): atomic write, then
  `synod fmt`.
- `list_files`, `read_buffer`, `get_diagnostics`, `get_selection` and
  `open_file` have `readOnlyHint: true`; the others have `readOnlyHint: false`
  and `destructiveHint: false`.
- On the bundled examples the read tools work and `apply_edits`,
  `replace_buffer`, `create_file` and `save_files` return `read-only: bundled
  examples; copy them to a folder first`.
- During a prompt-bar request the window's buffers are read-only to the user, and
  Claude's edits appear live. An external session takes no lock, so a `version`
  check fails Claude's edit if the user typed since its read. While a
  prompt-bar request runs in a window, write tools of an external session on that
  window return `busy`. Every call is serialized on the main actor through the
  editor model.

**Logging.** One `os_log` entry per call (subsystem `io.github.synodlang.synod`,
category `mcp`): kind of session (prompt bar or external), tool, path, byte
counts, ok or error code and duration. Buffer text, edit text and tokens are
never logged, and there are no log files.

## External sessions

Settings, Claude has "Let Claude Code in Terminal use the editor". It is off at
every launch and never persisted. While it is on, the listener runs and accepts
the external token, which is bound to the window whose module root equals the
`workspace` of the handshake; if no window has that root the handshake is
refused. "Copy MCP config" copies one line:

```
claude mcp add --scope local synod-editor -e SYNOD_EDITOR_TOKEN=<token> -- "<app>/Contents/Helpers/synod-editor-mcp" --socket "<sock>" --workspace "<module root>"
```

A note under the button says the line is valid until Synod quits or Disconnect
is pressed (a relaunch makes it stale, by design). The toolbar shows "Claude
(Terminal) connected" with Disconnect, which closes the connections and rotates
the token.

## `synod mcp`

`synod mcp [--root dir]` (compiler, `05-cli.md`) is a stdio server for agents
without a shell. It serves `doc`, `explain`, `check`, `run_summary` and `fmt`,
each exactly the CLI command it names, run in-process from the root, and has no
network code (decision 101). Any agent can use it with `claude mcp add synod --
synod mcp`. The prompt bar uses it instead of general shell access.

## Budget rules

- Never paste whole traces or scenes into a prompt.
- Never ask the model for coordinates, colors or timings; looks decide those.
- One CLI session per request; reuse it for follow-up edits in the same window.
