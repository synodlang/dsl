# 1. Lexical structure

## 1.1 Source files

- A source file has the extension `.synod`, is UTF-8, uses LF line endings and ends with a newline.
- A package is a folder; every `.synod` file in that folder belongs to it (see `spec/06`).

## 1.2 Lines and indentation

- Indentation is exactly **2 spaces** per level. Tabs are an error.
- Indentation opens and closes blocks (INDENT / DEDENT tokens, as in Python).
- One statement per line. There is no line continuation.
- Blank lines are allowed between statements; the formatter normalizes them (see `spec/10`).

## 1.3 Comments

`#` followed by a space, or `#` at the end of a line, starts a comment that
runs to the end of the line. A comment may stand on its own line or follow
a statement after one space. (`#` followed directly by six hex digits is a
color literal, see 1.7.)

## 1.4 Words

```
word  = letter { letter | digit } ;
qword = word "." word ;            (* a qualified word: package.Name *)
```

- Letters are ASCII `a-z` and `A-Z`; digits are `0-9`. No underscores or hyphens.
- A word starting with an upper-case letter is **exported** (see `spec/06`). Capitalization has no other meaning.
- `.` appears only in qualified words and in import paths.

## 1.5 Numbers and durations

```
number   = [ "-" ] digits [ "." digits ] ;
duration = number ( "ms" | "s" ) ;      (* no space between number and unit *)
```

- There is one numeric type, `number`: a 64-bit IEEE-754 float.
- Durations have type `time`. `ms` and `s` are the only units.
- Canonical: `0.5` (never `.5`), no trailing zeros after the decimal point
  (`1.5`, never `1.50`), and the largest unit that gives a whole number
  (`1s`, `1500ms`, never `1000ms`).

## 1.6 Text

```
text = '"' { char | escape } '"' ;
escape = '\"' | '\\' | '\n' ;
```

Double quotes only. Text is immutable.

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
| `[` `]` | holes, inside `define` patterns |
| `:` | inside a hole, between name and type |
| `(` `)` | a nested phrase |
| `{` `}` | a list literal, or the empty list/map `{}` |
| `,` | between list items, and between field settings after `message … with` |
| `.` | qualification, and inside import paths |

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
use as define native main has set to when after deliver emit
if else each in result always draw for message with of item
is not at least most less more than and or plus minus times divided by
true false
```

(`at least`, `at most`, `less than`, `more than`, `divided by`, `is in`,
`is not`, `is not in` are multi-word operators built from these words.)
