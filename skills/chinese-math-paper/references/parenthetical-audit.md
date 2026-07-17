# Explanatory Parenthetical Audit

> Added 2026-07-12 after comparing our paper v3 against Ou (2007) and KK (2023).
> Both reference papers use ZERO explanatory parentheticals.

## What is an explanatory parenthetical?

A parenthetical that **explains or justifies a proof step in prose**, as opposed to:
- Math notation: `(E_{ij})`, `(a_k + b_k)`
- Range specifiers: `(3 ≤ k ≤ n-2)`
- Case labels: `(Case 1)`, `(upper bound)`
- Single-word qualifiers: `(possibly)`, `(bilinear)`, `(non-orthogonal)`

## Audit methodology

1. Extract all parentheticals from the LaTeX source.
2. Filter out math-only, labels, and one-word qualifiers.
3. Identify those with prose keywords: `using`, `only`, `via`, `since`, `note that`, `trivially`.
4. Compare against reference papers (Ou, KK) using the same filter.
5. Decision per parenthetical: **delete** (reader can infer), or **absorb** (fold into sentence).

## Findings (paper v3, 2026-07-12)

Our paper had 4 explanatory parentheticals across 5 pages. Ou (6 pages) and KK (12 pages) both had 0.

| # | Parenthetical | Location | Decision |
|---|---------------|----------|----------|
| 1 | `(using d₃=d₁, d₄=d₂ from above)` | prelim.tex proof | Delete — just established two lines prior |
| 2 | `(only the bracket-right term contributes, via [E₁ᵤ,Eᵤₙ]=E₁ₙ)` | image-restriction.tex case (ii) | Delete — computation within reader's competence |
| 3 | `(only c_k E₂ₙ interacts with E₁₂)` | endpoint.tex "Pair with E₁₂" | Delete — "contributes -c_k" is a complete assertion |
| 4 | `(via [E₁,ₙ₋₁,Eₙ₋₁,ₙ]=E₁ₙ)` | endpoint.tex "Pair with E_{n-1,n}" | Delete — "analogue of the previous computation" already tells reader what to do |

All 4 were deleted (not absorbed). None were structurally necessary.

## Decision rule

When a parenthetical explains how a computation works:
- If the computation is a **standard bracket identity** that the target reader (Lie algebra classification) can perform in their head → **delete**.
- If the computation is genuinely non-obvious AND the explanation is a complete sentence → consider absorbing into the preceding sentence without parentheses.
- Ou and KK never use parentheticals for proof-step justification. Neither should we.

## Verification

After deletion, run the audit script again — explanatory parenthetical count must be 0 in both EN and ZH versions.
