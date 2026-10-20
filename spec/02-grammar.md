# 2. Grammar

Synod is parsed in two stages:

1. **Structure.** The grammar below turns a file into lines, blocks and
   sentences. It knows nothing about meaning.
2. **Resolution.** Each sentence is matched against the phrases in scope:
   language words, the package's own phrases, and the exported phrases of
   imported packages (see `spec/03`).

## 2.1 Structure grammar

```ebnf
file     = { line } ;
line     = sentence [ block ] NEWLINE ;
block    = NEWLINE INDENT line { line } DEDENT ;
sentence = item { item } ;
item     = word | qword | value | hole | "(" sentence ")" | list ;
hole     = "[" word ":" type "]" ;          (* only inside a define pattern *)
type     = word | qword ;
value    = number | duration | text | color | "true" | "false" ;
list     = "{" [ sentence { "," sentence } ] "}" ;
```

An **item** is the unit a hole receives: one token, or one parenthesized
group, or one list literal. This is why matching is exact (see `spec/03`).

## 2.2 Language word grammar

Every language word is a built-in phrase. Its exact form, using `arg` for
"one item":

```ebnf
arg        = word | qword | value | "(" sentence ")" | list ;

(* structure *)
import     = "use" path [ "as" word ] ;
definition = "define" pattern ( block | "native" text ) ;
pattern    = { word | qword | hole } ;      (* rules in spec/03 *)
entry      = "main" block ;

(* state and records *)
typedecl   = word "has" block ;            (* block holds only assignments *)
assignment = "set" place "to" arg ;
place      = word | qword | field | element ;

(* events *)
handler    = "when" arg block ;
delay      = "after" arg block ;
delivery   = "deliver" arg "to" arg ;
emission   = "emit" arg ;

(* control *)
choice     = "if" arg block [ "else" block ] ;   (* else on its own line, same indent as if *)
loop       = "each" word [ "," word ] "in" arg block ;
return     = "result" arg ;

(* checking *)
invariant  = "always" arg ;

(* visual *)
drawing    = "draw" arg "as" word [ "for" arg ] [ block ] ;

(* values *)
record     = "message" type "with" setting { "," setting } ;
setting    = word arg ;
field      = ( word | qword ) "of" arg ;
element    = "item" arg "of" arg ;
compare    = arg ( "is" | "is not" | "is in" | "is not in"
                 | "less than" | "more than" | "at least" | "at most" ) arg ;
logic      = arg ( "and" | "or" ) arg | "not" arg ;
arithmetic = arg ( "plus" | "minus" | "times" | "divided by" ) arg ;
```

## 2.3 Consequences of the grammar

- **No precedence rules.** A hole takes one item, so every nested phrase
  is parenthesized: `((a plus b) times c)` is the only spelling of that
  meaning.
- **Conditions are parenthesized**, like every other multi-item argument:
  `if (Role is "leader")`, `always (Count less than 2)`. A single-item
  condition is bare: `if granted`.
- **A one-word phrase is written bare.** `time.Now`, `sim.Here`,
  `theme.Accent`, `Quorum` are single items and need no parentheses.
- **`else`** starts its own line at the same indentation as its `if`.
- **Binding in `when`.** Inside the event phrase of a `when`, a bare word
  that is not yet a name becomes a new name bound for the block, and a
  type name binds the matched record under that type's name:
  `when (net.Received digest from sender)` binds `digest` and `sender`.
- **Locals.** `set x to …` where `x` is not declared state creates a local
  for the rest of the enclosing block, including nested blocks.
- **`else if`** does not exist; nest an `if` inside `else`.
