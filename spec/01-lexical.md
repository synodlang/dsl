# 1. Lexical structure

## 1.1 Source files

- A source file has the extension `.synod`, is UTF-8, uses LF line endings and ends with a newline.
- A byte order mark at the start and CR LF line endings are accepted when reading and are `non-canonical`; `synod fmt` removes them. Invalid UTF-8 and a CR not followed by LF are `syntax` errors.
- A package is a folder; every `.synod` file in that folder belongs to it (see `spec/06`).

## 1.2 Lines and indentation

- Indentation is exactly **2 spaces** per level. A tab in indentation is an `indent` error; a tab between tokens is a `syntax` error. Tabs are allowed inside comments.
- Indentation opens and closes blocks (INDENT / DEDENT tokens, as in Python).
- One sentence per line; a sentence never continues on the next line. A statement's blocks and continuation lines (`else`, see `spec/02` §2.1) follow it on their own lines.
- Blank lines are allowed between statements and between a block and its continuation line; the formatter normalizes them (see `spec/10`).
- A line holding only spaces is a blank line.

## 1.3 Comments

`#` followed by a space, or `#` at the end of a line, starts a comment that
runs to the end of the line. A comment may stand on its own line or follow
a statement after one space. Everything after the first space is the
comment's text, kept verbatim; a bare `#` is an empty comment.

- `#` followed directly by six hex digits and then a character that is not a letter or digit is a color literal (§1.7). Any other `#` (`#todo`, `#ABC`) is a `syntax` error.
- A comment line is not code: its indentation is not checked, so it never opens or closes a block (the formatter re-indents it).
- Comments belong to the line they stand on or the line that follows them; they never form blocks or statements (`spec/02` §2.1).

## 1.4 Words

```
word   = letter { letter | digit } ;
member = word "." word { "." word } ;   (* one token, no spaces around "." *)
```

- Letters are ASCII `a-z` and `A-Z`; digits are `0-9`. No underscores or hyphens.
- A word starting with an upper-case letter is **exported** (see `spec/06`). Capitalization has no other meaning.
- `.` joins the words of a member, which names a named member (`spec/03` §3.3). `.` also appears in numbers (`0.5`) and in import paths.
- Every word starts with a letter, so field names never start with a digit and `nodes.0` is not a member. Elements are written `item 0 of nodes`.
- Two tokens that are words, members, numbers, durations, texts or colors are always separated by whitespace; `x-1`, `5-3` and `"a""b"` are `syntax` errors. Punctuation needs no space to be lexed, but the formatter spaces it (`spec/10`).

## 1.5 Numbers and durations

```
number   = [ "-" ] digits [ "." digits ] ;
duration = number ( "ms" | "s" ) ;      (* no space between number and unit *)
```

- There is one numeric type, `number`: a 64-bit IEEE-754 float.
- Durations have type `time`. `ms` and `s` are the only units.
- Canonical: `0.5`, no trailing zeros after the decimal point
  (`1.5`, never `1.50`), no leading zeros (`7`, never `007`), and the largest unit that gives a whole number
  (`1s`, `1500ms`, never `1000ms`). A fractional millisecond count stays in milliseconds (`0.5ms`).
- `.5`, a leading `+` and spaces around `.` are never written: each is a `syntax` error, not something the formatter fixes. `-` directly before a digit is part of the number; subtraction is `minus`. There are no exponents.

## 1.6 Text

```
text = '"' { char | escape } '"' ;
escape = '\"' | '\\' | '\n' ;
```

Double quotes only. Text is immutable. Text holds no control characters (U+0000 to U+001F, U+007F, tabs included); write `\n` for a line break. An unknown escape or an unterminated text is a `syntax` error.

## 1.7 Colors

```
color = "#" hex hex hex hex hex hex ;
```

Color literals are allowed **only** as the argument of `result` in a
phrase with no holes, which names the color (see `spec/09`). Everywhere
else colors are named: `theme.Accent`.

## 1.8 Punctuation

| Token | Only use |
| --- | --- |
| `[` `]` | holes: in `define` patterns (receivers included) and in `when` (receivers and typed event bindings) |
| `:` | inside a hole, between name and type |
| `(` `)` | a nested sentence |
| `{` `}` | a list literal, or the empty list/map `{}` |
| `,` | between list items, between settings after `message … with`, and between the two names after `each` |
| `.` | members: `net.Send`, `n.Known`, `Vote.term`; also inside numbers and import paths |

## 1.9 Paths

Import paths appear only after `use` and are lexed as a single token:

```
path    = segment { "/" segment } [ "@" version ] ;
segment = ( letter | digit ) { letter | digit | "." | "-" } ;
version = "v" digits "." digits "." digits ;
```

`std/net`, `github.com/synodlang/examples/raft`, `github.com/someone/raft@v1.2.0`.

## 1.10 Reserved words

The following words are language words and cannot be used as names:

```
use as define native main start has set to when after deliver emit
if else each in result always draw for message with of item list map
is not at least most less more than and or plus minus times divided by
true false
```

(`at least`, `at most`, `less than`, `more than`, `divided by`, `is in`,
`is not`, `is not in` are multi-word operators built from these words.)

`main` and `start` appear only in the entry definitions `define main` and `define [n: Node] start`. `list` and `map` appear only in types (`list of node`, `map of text to number`).
