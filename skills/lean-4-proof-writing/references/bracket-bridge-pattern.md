# Bracket Bridge Pattern — Architecture-first proof development

## The Anti-Pattern: Scattered Sorries

When a theorem requires 16-case `split_ifs` and 10+ sorries scattered across
branches, the proof structure is wrong. Each sorry represents the SAME
mathematical gap expressed in different index configurations.

## The Pattern: Identify Independent Generating Mechanisms

Two mechanisms that generate different classes of coefficient identities:

1. **`half_deriv_cond`** (splitting geometry): decomposes a source along its bracket
   structure. Controls coefficients aligned with the source's own indices.
   ```
   2·coeff(D(E_ij), target) = splitting of E_ij at any k
   ```

2. **`coeffOf_cond_zero`** (commuting geometry): the bracket identity for two
   DISJOINT sources. Allows coefficient information to flow transversely —
   from source index `i` to target index `u` (with u ≠ i).
   ```
   [D(E_ij), E_cd] + [E_ij, D(E_cd)] = 0   when [E_ij, E_cd] = 0
   ```

These are **independent**. Neither subsumes the other (verified by Jacobi reduction
audit — Jacobi preserves the left source index, cannot achieve transverse propagation).

## The Solution: Semantics Bridge Layer

Create a bridge file that expresses both mechanisms in a unified coefficient form:

```lean4
-- Bracket coefficient extraction (pure combinatorics, no half-deriv needed)
noncomputable def bracketLeft (D : HalfDerivation F n) (i j c d u v : ℕ) : F :=
  (if u < c ∧ v = d then coeffOf D i j u c else 0)    -- "A"
  - (if u = c ∧ d < v then coeffOf D i j d v else 0)  -- "B"

noncomputable def bracketRight (D : HalfDerivation F n) (i j c d u v : ℕ) : F :=
  (if u = i ∧ j < v then coeffOf D c d j v else 0)    -- "C"
  - (if u < i ∧ v = j then coeffOf D c d u i else 0)  -- "D"
```

Then `coeffOf_cond_zero` becomes: `bracketLeft + bracketRight = 0`.

When source is non-adjacent (j-i ≥ 2), use `coeffOf_nonadjacent` to kill individual terms.
When source is adjacent (j = i+1), the gap converges to a single lemma rather than
10+ scattered sorries.

## Architecture

```
Infrastructure (MatIdx, HalfDerivation)
      │
Bracket (coeffOf, coeffOf_cond, coeffOf_nonadjacent)
      │
Semantics (bracketLeft/bracketRight + half_deriv_bracket_zero)
      │
BracketCondZero (coeffOf_cond_zero, refactored to 2 by_cases)
      │
ToNCoeff → Adjacent → Classification
```

All dependencies strictly unidirectional. No import cycles.

## Key Insight (from Jacobi Reduction Audit)

> Jacobi identity preserves the left source index along bracket chains.
> It cannot achieve transverse coefficient propagation (source index i → target index u ≠ i).
> Therefore `coeffOf_cond_zero` (commuting identity) is a genuinely independent axiom.
> The adjacent-source gap cannot be bypassed by algebraic reduction.

See `ARCHITECTURE-AUDIT.md` in the project root for the full analysis.

## `split_ifs` Pitfall

`split_ifs with hA hB` creates `hA : P → False` for the false branches (NOT `¬P`).
`rcases` can't destruct function types. Use bare `split_ifs` with `rename_i`:

```lean4
split_ifs
· rename_i hA hB; rcases hA with ⟨huc, hvd⟩; ...  -- both true
· rename_i hA; rcases hA with ⟨huc, hvd⟩; ...      -- only first true
```

Or manually derive `¬(A ∧ B)` via `intro h; rcases h with ...; omega`.
