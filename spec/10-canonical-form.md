# 10. Canonical form

The compiler rejects non-canonical code with `non-canonical` errors.
`synod fmt` rewrites any parseable file into canonical form, so in
practice nobody formats by hand. Code across all repositories looks alike.

| Area | Canonical rule |
| --- | --- |
| Encoding | UTF-8, LF, final newline, no trailing spaces |
| Header | the first line is `# package <name>` |
| Imports | directly after the header, one per line, sorted by path, no blank lines between them |
| Indentation | 2 spaces per level |
| Blank lines | exactly one between top-level statements (none between imports); none inside blocks |
| Comments | `# ` with one space; a trailing comment has one space before `#` |
| Parentheses | around every multi-item argument; never around a single item |
| Holes | always typed: `[n: node]` |
| Numbers | `0.5`, `-1`, `2`; no leading `+`, no trailing zeros |
| Durations | the largest unit giving a whole number: `1500ms`, `2s` |
| Text | double quotes; escapes only `\"`, `\\`, `\n` |
| Lists | `{a, b}`: one space after each comma, none inside the braces |
| Records | `message T with a 1, b 2`: settings in declaration order of the fields |
| Colors | upper-case hex: `#F4A340` |
| `else` | on its own line, at its `if`'s indentation |
| Negation | `not (a is b)` becomes `(a is not b)`; `not (a less than b)` becomes `(a at least b)`; likewise for `more than`, `at least`, `at most`, `is in` |
| Names | exported names start upper-case, nothing else does; no underscores |

The negation row is how Synod keeps one spelling per meaning while still
offering `is not`, `at least` and friends: the formatter always picks the
direct operator.
