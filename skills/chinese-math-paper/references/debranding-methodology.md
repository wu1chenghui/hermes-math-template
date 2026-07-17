# Debranding Methodology — Removing Self-Coined Terms

> Principle: Write like KK and Ou — no named techniques, no metaphors for
> indices, no "Observation" as a proper noun. Use equation/lemma numbers.

## What constitutes a self-coined term

A term is self-coined if it appears as a **named concept used throughout the
paper** but is NOT standard mathematical vocabulary. Three classes:

1. **Named proof techniques**: Giving a proper name to an equation or lemma
   (e.g., "chain bridge", "width reduction", "diagonal propagation").
2. **Index metaphors**: Renaming standard matrix indices with new words
   (e.g., "source (i,j)" instead of plain "(i,j)", "target (u,v)" instead
   of "the (u,v)-coordinate", "width" for j-i).
3. **Capitalized observations**: Formatting a simple fact as a named entity
   (e.g., "Observation." → "Observe that...").

## Detection method

1. Search KK/Ou papers for the same concept — if they don't name it, we shouldn't.
2. Search for capitalized mid-text names, subsection titles that name techniques,
   and any word used as a shorthand that could be replaced by a lemma/equation number.

## Fixing method

| Problem | Fix |
|---------|-----|
| Named technique used inline | Replace with `\eqref{...}` or `Lemma~\ref{...}` |
| Named subsection title | Replace with descriptive phrase |
| Index metaphor | Write indices directly in math notation |
| Capitalized fact | Lowercase and integrate into paragraph flow |

## Example: our paper v3 debranding

| Before | After |
|--------|-------|
| "the chain bridge" | `\eqref{eq:chainbridge}` |
| "by width reduction (Lemma 1.1)" | "By Lemma 1.1" |
| §1.4 "Diagonal propagation" | "Equality of diagonal entries" |
| "a-channel / c-channel" | "a_k-coefficient / c_k-coefficient" |
| "the Observation" | "Observe that..." |
| "source (i,j)" | "(i,j)" |
| "target (u,v)" | "(u,v)" |
| "width j-i-1" | "j-i" or "j = i+2" |
