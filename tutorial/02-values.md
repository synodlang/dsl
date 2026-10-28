# 2. Values

## Goal

Show a sentence the program computes.

## New in this chapter

<!-- new -->
- Locals: `set price to 4` ([spec/02 §2.3](../spec/02-grammar.md))
- Numbers ([spec/01 §1.5](../spec/01-lexical.md)) and the type `number` ([spec/04 §4.1](../spec/04-types-and-values.md))
- `plus`, `minus`, `times`, `divided by` ([spec/04 §4.5](../spec/04-types-and-values.md))
- Parentheses: one item per hole ([spec/02 §2.3](../spec/02-grammar.md))
- Lists `{…}` and the type `list of T` ([spec/04 §4.2](../spec/04-types-and-values.md), [§4.4](../spec/04-types-and-values.md))
- `text.From` and `text.Join` from `std/text` ([std/README.md](../std/README.md))

## Steps

We start from the last program of chapter 1. A local is a name for a value: `set price to 4` creates `price`. Showing a number needs a conversion, because a caption is text; `text.From` turns any value into text. A phrase with a value inside another sentence goes in parentheses:

<!-- step 1 program=hello -->

Locals can be computed from other locals. `times` is one of four arithmetic words, written between its two values:

<!-- step 2 program=hello -->

Showing "13" alone is not much of a sentence. `text.Join` glues a list of texts together. A list is written in braces, items separated by commas, and each item is one thing: a text, or a parenthesized sentence such as `(text.From total)`:

<!-- step 3 program=hello -->

Now the whole bill. `plus`, `minus` and `divided by` work like `times`. An operator takes exactly two items, so a longer calculation nests parentheses, and there are no precedence rules to learn:

<!-- step 4 program=hello -->

## What to watch

Three captions in order: the greeting, `Total: 13` and `Each pays 5.5`. Everything happened at the very first instant; the computation is not drawn, only its result.

## Why it looks like this

A local that nobody reads is almost always a mistake, so Synod refuses it. Here `coupon` is set and never used:

<!-- why unused-name -->

The diagnostic is `unused-name`. Use the value, or delete the line ([spec/02 §2.3](../spec/02-grammar.md)).

## Exercises

1. Change `tip` to 3. What does each person pay?
<details><summary>Expected</summary>

The total is 15 and each pays 6.5.

</details>

2. Write `(apples times price plus tip)` and run `synod check hello`. What happens?
<details><summary>Expected</summary>

It is rejected: an operator takes two items, so you must write `((apples times price) plus tip)`.

</details>

3. Add `lesson.Note (text.Join {"People: ", (text.From people)})`.
<details><summary>Expected</summary>

A fourth caption, `People: 2`.

</details>
