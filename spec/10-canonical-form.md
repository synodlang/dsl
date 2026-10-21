# 10. Canonical form

The compiler rejects non-canonical code with `non-canonical` errors.
`synod fmt` rewrites any parseable file into canonical form, so in
practice nobody formats by hand. Code across all repositories looks alike.

| Area | Canonical rule |
| --- | --- |
| Encoding | UTF-8, LF, final newline, no trailing spaces |
| Header | the first line is `# package <name>`: the folder name, or `main` in the folder's main file |
| Imports | directly after the header, one per line, sorted by path, no blank lines between them |
| Indentation | 2 spaces per level |
| Blank lines | exactly one between top-level declarations (none between imports); none inside blocks |
| Comments | `# ` with one space; a trailing comment has one space before `#` |
| Parentheses | around every multi-item argument; never around a single item |
| Holes | always typed, element types included: `[n: node]`, `[ns: list of Node]` |
| Receivers | `define [n: Node] Learn [k: node] at [v: number]`, `when [n: Node] (net.Received digest from sender)`: one space after the receiver |
| Members | `n.Known`, `Vote.term`: no spaces around `.`, never parenthesized |
| Numbers | `0.5`, `-1`, `2`; no leading `+`, no trailing zeros |
| Durations | the largest unit giving a whole number: `1500ms`, `2s` |
| Text | double quotes; escapes only `\"`, `\\`, `\n` |
| Commas | `{a, b}`, `message T with a 1, b 2`, `each k, v in m`: one space after each comma, none before, none inside braces |
| Records | `message T with a 1, b 2`: settings in declaration order of the fields |
| Colors | upper-case hex: `#F4A340` |
| Continuation lines | `else`, `otherwise`: own line at the statement's indentation, directly after its block |
| Negation | `not (a is b)` becomes `(a is not b)`; `not (a less than b)` becomes `(a at least b)`; likewise for `more than`, `at least`, `at most`, `is in` |
| Names | exported names start upper-case, nothing else does; no underscores |

The negation row is how Synod keeps one spelling per meaning while still
offering `is not`, `at least` and friends: the formatter always picks the
direct operator.
