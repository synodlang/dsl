# 2. Grammar

Synod is parsed in two stages:

1. **Structure.** The grammar below turns a file into lines, blocks and
   sentences. It knows nothing about meaning.
2. **Resolution.** Each sentence is matched against the language forms
   and the phrases in scope: the package's own phrases and the exported
   phrases of imported packages (see `spec/03`).

## 2.1 Structure grammar

```ebnf
file         = { line } ;
line         = sentence [ block { continuation } ] NEWLINE ;
block        = NEWLINE INDENT line { line } DEDENT ;
continuation = word block ;                  (* a continuation line, see below *)
sentence     = items { "," items } ;
items        = item { item } ;
item         = word | member | value | hole | "(" sentence ")" | list ;
hole         = "[" word ":" type "]" ;      (* in define patterns and in when *)
type         = word | word "." word
             | "list" "of" type
             | "map" "of" type "to" type ;
value        = number | duration | text | color | "true" | "false" ;
list         = "{" [ item { "," item } ] "}" ;
```

- **Comments and layout.** Comments are not part of the grammar above:
  each attaches to the line that follows it (or to the line it trails) and
  never forms a block or a statement. A file may start with the package
  header (`spec/06` §6.1), which is also a comment. `use` lines stand only at
  the top level (`use` nested in a block is a `syntax` error), and `()` is a
  `syntax` error.
- An **item** is the unit an inline hole receives: one token, one
  parenthesized sentence, or one list literal. This is why matching is
  exact (see `spec/03`).
- A **member** (`spec/01` §1.4) is one item: `n.Known`, `Vote.term`,
  `server.tcp.State`, `n.Version`.
- **Commas.** Commas stand only between list items, between
  `message … with` settings and between the two `each` names (§2.2); a
  comma anywhere else is a `syntax` error.
- **Continuation lines.** A line that holds a single bare word and has a
  block, at the same indentation as the statement before it and directly
  after that statement's block (no line in between), is a continuation
  line. Its word and block join that statement's sentence as its next word
  and block hole. `else` is the continuation line of `if`; `otherwise` is
  the continuation line of `Retry … times` (`spec/03` §3.1). A
  continuation line that follows no block is a `syntax` error. Blank and
  comment lines may stand between a block and its continuation line. No
  statement is a single word with a block (`spec/03` §3.1, rule 5), so
  every such line is a continuation line.

## 2.2 Language word grammar

Every language word belongs to a built-in form. Its exact shape, using
`arg` for "one item that is not a hole":

```ebnf
arg        = item ;

(* files and entries *)
import     = "use" path [ "as" word ] ;
definition = "define" pattern ( block | "native" text ) ;
pattern    = [ hole ] head { word | hole } ;     (* first hole: the receiver; rules in spec/03 *)
head       = word ;                              (* never a reserved word *)
main       = "define" "main" block ;
start      = "define" hole "start" block ;       (* hole: the receiver [n: Node] *)

(* state and records *)
typedecl   = word "has" block ;                  (* block holds only "set" word "to" arg lines *)
assignment = "set" place "to" arg ;
place      = word | member | "item" arg "of" ( word | member ) ;

(* events *)
handler    = "when" [ hole ] arg block ;         (* the receiver: present exactly on node events *)
delay      = "after" arg block ;
delivery   = "deliver" arg "to" arg ;
emission   = "emit" arg "by" arg ;               (* the one form of emit; spec/05 *)

(* control *)
choice     = "if" arg block [ "else" block ] ;   (* else is a continuation line *)
loop       = "each" word [ "," word ] "in" arg block ;
return     = "result" arg ;

(* checking *)
invariant  = "always" arg ;

(* visual *)
drawing    = "draw" arg "as" word [ "for" arg ] [ block ] ;
attachment = word "of" arg ;                     (* look context: a shape attached with draw … for *)

(* values *)
record     = "message" ( word | word "." word ) "with" setting { "," setting } ;
setting    = word arg ;
element    = "item" arg "of" arg ;
compare    = arg ( "is" | "is not" | "is in" | "is not in"
                 | "less than" | "more than" | "at least" | "at most" ) arg ;
logic      = arg ( "and" | "or" ) arg | "not" arg ;
arithmetic = arg ( "plus" | "minus" | "times" | "divided by" ) arg ;
```

- `Node has` declares the fields of the package's node type (`spec/05`);
  `T has` with any other word `T` declares the record type `T` (`spec/04`).
- Members are items, not language forms. `spec/03` §3.3 says what each
  one names: a package member (`net.Send`, `gossip.Node`), a record field
  (`Vote.term`, `api.Get.Key`), a node field (`n.Known`,
  `server.tcp.State`) or a receiver phrase (`n.Version`, `n.Learn k at v`).

## 2.3 Consequences of the grammar

- **No precedence rules.** An inline hole takes one item, so every nested
  phrase is parenthesized: `((a plus b) times c)` is the only spelling of
  that meaning.
- **Conditions are parenthesized**, like every other multi-item argument:
  `if (n.Role is "leader")`, `always (Count less than 2)`. A single-item
  condition is bare: `if grant`, `if Vote.granted`.
- **A zero-hole phrase is one item**, written bare: `time.Now`,
  `theme.Accent`, `Quorum`, `raft.Leaders`, `n.Version`.
- **Members are never parenthesized:** `n.Known`, `Vote.term`.
- **Parentheses are removed, never added, by the formatter.** It unwraps
  a parenthesized single item but never wraps a multi-item argument; a
  missing pair is reported by the compiler (`unknown-phrase`).
- **Entries.** `define main` and `define [n: Node] start` are language
  forms; `main` and `start` are reserved words and never phrase heads.
- **`else`** is a continuation line: its own line, at its `if`'s
  indentation, directly after the `if` block. `else if` does not exist;
  nest an `if` inside `else`.
- **Binding in `when`.** `when [n: Node] (net.Received digest from sender)`
  binds the receiver `n`, the record `digest` and the peer `sender` for
  the block (`spec/05` §5.6).
- **Locals.** `set x to …` with a bare word `x` creates a local for the
  rest of the enclosing block, nested blocks included, or updates it.
  Locals are **block scoped**: one created inside an `if` branch, an
  `else` branch, an `each` body or another block is not visible after
  that block, so a value needed afterwards is set before the block.
  Hole parameters, `when` bindings and `each` names cannot be set
  (`bad-place`), nor can a local from outside an `after` block or a block
  argument be set inside it (`bad-place`; the block holds a copy, `spec/07` §7.4).
  A local is **used** when it is read, or is the base of `set item … of x`;
  setting it again does not use it (`unused-name`).
