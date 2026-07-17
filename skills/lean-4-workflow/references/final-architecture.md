# Project Architecture (2026-06-19) — HalfDerivation Release Candidate

Final architecture for the 1/2-derivation classification on N_n.

## Dependency DAG (main line)

```
MatrixBasis → BracketFormula → ProjectionIdentity (Common)
                                     │
                               HalfDerivation
                                     │
                              ReductionTree (pure combinatorics)
                                     │
                              Evaluation (algebraic, char≠2)
                                     │
                              Leaf/Adjacent (confluence → all_adjacent_equal)
                                     │
                              HalfNonadjacent (offdiag vanishing)
                                     │
                              HalfClassification (final assembly)
```

## Module Map

| Layer | File | Theorems | sorry |
|-------|------|----------|-------|
| Reduction | `E/Reduction/ReductionTree.lean` | 4 | 0 |
| Evaluation | `E/Reduction/Evaluation.lean` | 5 | 0 |
| Leaf | `E/Leaf/Adjacent.lean` | 3 | 0 |
| Classification | `E/Classification/HalfClassification.lean` | 3 | 0 |

## Key Theorems

- `width_strictly_decreases` — child width < parent width (pure ℕ)
- `eval_diagonal` — 2*coeff(i,j;i,j) = coeff(i,k;i,k) + coeff(k,j;k,j)
- `eval_diagonal_invariant` — same child sum for ANY split (CONFLUENCE)
- `skip_two_eq` — coeff(i,i+1;i,i+1) = coeff(i+2,i+3;i+2,i+3)
- `all_adjacent_diag_equal` — all adjacent-diagonal equal (n≥5)
- `all_diag_equal` — all diagonal coeffs = scalar (induction on width)

## Build

lake build → 2967 jobs ✅

## Shared Layer (ProjectionIdentity)

Only.lean + Pair.lean parameterized by `ProjectionIdentity F` — one proof shared
by both NnDerivation and HalfDerivation via `toProjectionIdentity`.
