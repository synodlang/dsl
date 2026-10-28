# 1. Hello

## Goal

Show a sentence on the stage.

## New in this chapter

<!-- new -->
- The file header `# package main` ([spec/06 §6.1](../spec/06-packages-and-programs.md))
- Comments, blocks and indentation ([spec/01 §1.2](../spec/01-lexical.md), [§1.3](../spec/01-lexical.md))
- `use` and text values ([spec/06 §6.3](../spec/06-packages-and-programs.md), [spec/01 §1.6](../spec/01-lexical.md))
- `define main`, the program's start ([spec/06 §6.5](../spec/06-packages-and-programs.md)), and the main context ([spec/05 §5.4](../spec/05-state-and-contexts.md))
- `lesson.Note` from `std/lesson` ([std/README.md](../std/README.md))
- Canonical form and `synod fmt` ([spec/10](../spec/10-canonical-form.md))

## Steps

Make a folder `hello` and put a file `hello.synod` in it. Every step is that one file.

Synod has no setup to learn first. A program is a folder, and the folder's start is a file whose first line says it is the program:

<!-- step 1 program=hello -->

Line 1 is the header: `# package main` says "this file is the folder's program". Line 2 is a `use`: it brings in `std/lesson`, a small package from the standard library that talks to the viewer. Then `define main` says what happens when the program starts. Its body is the indented line below it, a block, and it is a sentence: `lesson.Note` followed by the text `"Hello, Synod"`. Run it with `synod run hello`.

A line that starts with `# ` is a comment. It belongs to the line below it, so a comment above `define main` describes the program:

<!-- step 2 program=hello -->

A block holds as many sentences as you like, one per line, all indented the same:

<!-- step 3 program=hello -->

## What to watch

The stage is empty, and a caption appears: first the greeting, then the second sentence. Nothing else exists yet, so there is nothing to draw but the words. Each `lesson.Note` is one caption, shown in the order you wrote them.

## Why it looks like this

Indentation is two spaces per level, and it is the only way to say that lines belong together. A tab is not a different kind of space; it is an error you can see at once:

<!-- why indent -->

The diagnostic is `indent`. For every other detail of layout (blank lines, spacing, the order of `use` lines) you never have to remember a rule: `synod fmt hello` rewrites the file into the one canonical form, and `synod fmt --check hello` tells you if a file differs. Synod has one way to write each thing, so code looks alike everywhere ([spec/10](../spec/10-canonical-form.md)).

## Exercises

1. Add a third `lesson.Note` line. How many captions do you see?
<details><summary>Expected</summary>

Three, in the order written.

</details>

2. Delete the `use std/lesson` line and run `synod check hello`. What does it say?
<details><summary>Expected</summary>

`lesson.Note` is an unknown phrase: a package must be imported before you use its phrases.

</details>

3. Indent one `lesson.Note` line by three spaces and run `synod fmt --check hello`, then `synod check hello`.
<details><summary>Expected</summary>

The indentation is not a multiple of two spaces, so the file is rejected. Indent by two spaces per level and it passes.

</details>
