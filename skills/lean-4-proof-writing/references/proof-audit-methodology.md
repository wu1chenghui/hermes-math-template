# Proof Audit Methodology (Multi-Sorry Triage)

When facing a file with **multiple `sorry`'d lemmas** that all block one main
theorem, do NOT jump into proving individual lemmas. Perform a **proof audit**
first.

## Step 1: Get goal context for each sorry

Use LSP tools to inspect each `sorry`:
- `lean_term_goal` with column position — works when `lean_goal` returns empty
  (which happens on `by sorry` lines where the sorry closes the goal)
- `lean_hover_info` on key lemmas used in the file — check type signatures

## Step 2: Map available APIs

For each lemma, list which existing APIs could apply. Check:
- Does this lemma's source have a split point (`i < k < j`)?
- Is the source non-adjacent (`j - i ≥ 2`)?
- Does the target differ from the source diagonal (`u ≠ i ∨ v ≠ j`)?

Common API gaps:
- `coeffOf_nonadjacent` **has a sorry for adjacent sources** — don't rely on it
  for `j = i+1`
- `coeffOf_cond` needs `i < k < j` — useless when `j = i+1`
- The bracket identity only handles non-adj sources

## Step 3: Build a difficulty/dependency table

Classify each lemma by difficulty (⭐ to ⭐⭐⭐⭐):

- **⭐⭐**: Target has one index mismatch with source. Embedding in a non-adjacent
  source (typeA/typeB variants) likely works.
- **⭐⭐⭐**: Two sources overlap (e.g., `c = i`), creating index entanglement.
- **⭐⭐⭐⭐**: Two adjacent sources simultaneously. This is the N₃ core — bracket
  identity is genuinely independent of half_deriv_cond here.

## Step 4: Identify degeneration paths

Some lemmas reduce to simpler ones via existing APIs:
- "pair" lemmas (two terms) → "only" lemmas (one term) when the second term
  vanishes via an existing lemma applied to a non-adjacent source
- Complex cases (*when both sources are adjacent*) may be the true minimal obstruction

## Step 5: Identify the minimal obstruction

The hardest lemma that **cannot** be reduced to simpler ones is the **minimal
obstruction**. Strategy: prove easier lemmas first, then attack the minimal
obstruction with a focused paper+Lean approach.

## Lean tools useful for auditing

| Tool | Use |
|------|-----|
| `lean_term_goal` | Get expected type when `lean_goal` returns empty |
| `lean_hover_info` | Check type signature of an existing lemma |
| `lean_diagnostic_messages` | See errors in a file |
| `lean_file_outline` | Get all declarations + type signatures |

## Pitfalls

- `lean_multi_attempt` replaces the ENTIRE target line with the snippet.
  It won't work on `by sorry` lines (breaks syntax). Use only on lines
  that are valid tactic blocks.
- `lean_goal` returns empty goals on `by sorry` lines because `sorry` closes
  the goal. Use `lean_term_goal` with a column position instead.

## Example: Auditing AdjacentBridge.lean (June 2026)

Real-world example of this methodology applied to 8 sub-lemmas:

| Lemma | Structure | Degeneration | Difficulty |
|-------|-----------|-------------|------------|
| `only_A` | target 2nd != source 2nd | typeA embedding variant | ⭐⭐ |
| `only_B` | target 1st != source 1st | typeB embedding variant | ⭐⭐ |
| `pair_cancel_AC` | two-term, reduces to only_B | if (c,d) nonadj -> 2nd term=0 | ⭐⭐ |
| `pair_cancel_AD` | two-term, reduces to only_A | if (c,d) nonadj -> 2nd term=0 | ⭐⭐ |
| `pair_cancel_BC` | c=i, sources overlap | if d>i+1 -> (c,d) nonadj | ⭐⭐⭐ |
| `pair_cancel_BD` | symmetric to BC | same | ⭐⭐⭐ |
| `only_C` | source (c,d) can be adjacent | nonadj -> trivial; adj -> hard | ⭐⭐⭐⭐ |
| `only_D` | symmetric to C | same | ⭐⭐⭐⭐ |

**Minimal obstruction**: `only_C` when (c,d) is adjacent (`d = c+1`).
This is the exact N₃ core case where bracket identity is independent of
half_deriv_cond.
