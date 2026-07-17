# Reduction Tree Theory — Complete Pipeline (2026-06-19)

The project discovered a unified reduction framework for (1/2)-derivation classification.
This document captures the full pipeline from combinatorial tree to leaf evaluation.

## Pipeline Overview

```
MatrixBasis → BracketFormula → ProjectionIdentity (Common Layer)
                                      │
                        ┌─────────────┴─────────────┐
                        ↓                           ↓
                  NnDerivation               HalfDerivation
                        │                           │
                        │              ┌────────────┴────────────┐
                        │              ↓            ↓            ↓
                        │        ReductionTree  Evaluation  Leaf/Adjacent
                        │              │            │            │
                        │         (combinatorial) (algebraic)  (leaf equality)
                        │
                  Applications
                  (Chain, Bridge, Diagonal)
```

## Layer 1: ReductionTree (purely combinatorial)

File: `E/Reduction/ReductionTree.lean`
Imports: ONLY `Mathlib.Tactic` (zero project dependencies)

Key theorems:
- `width_strictly_decreases` — every coeffOf_cond child has strictly smaller source width
- `reduction_terminates` — all paths finite (width as decreasing measure on ℕ)
- `leaf_iff_adjacent` — node is leaf iff source width=1
- `all_maximal_nodes_adjacent` — all maximal nodes are adjacent sources

## Layer 2: Evaluation (algebraic, HalfDerivation-specific)

File: `E/Reduction/Evaluation.lean`
Imports: HalfDerivation, HalfProjection, ReductionTree

Key theorems:
- `eval_diagonal` — 2·coeff(i,j;i,j) = coeff(i,k;i,k) + coeff(k,j;k,j) for any split k
- `eval_diagonal_half` — parent = (1/2)·(left + right) using CharNeTwo
- **`eval_diagonal_invariant`** — child sum independent of split k (CONFLUENCE)
- `eval_edge_weight` — each child gets weight 1/2
- `eval_nonadjacent_expand` — expand width≥2 node at split i+1

## Layer 3: Leaf Theory (algebraic)

File: `E/Leaf/Adjacent.lean`
Imports: HalfDerivation, ReductionTree, Evaluation

Key theorem:
- **`skip_two_eq`** — coeff(i,i+1;i,i+1) = coeff(i+2,i+3;i+2,i+3)
  Proof uses Confluence on E(i,i+3) + diagonal evaluation + ring algebra in field F

## Key Proof Technique: Confluence + Ring Algebra

The `skip_two_eq` proof is the canonical example of combining:
1. Confluence (`eval_diagonal_invariant`) on nested sources
2. Diagonal evaluation (`eval_diagonal`) on middle terms
3. Ring algebra in field F (using `ring`, not `linarith` which fails on Field F)

Pattern:
```lean
  -- Step 1: Confluence gives A+X = Y+B
  have h_conf := eval_diagonal_invariant D i (i+3) k₁ k₂ ...
  
  -- Step 2: Diagonal evaluation gives 2X = C+B, 2Y = A+C
  have h_X : 2 * X = C + B := eval_diagonal D ...
  have h_Y : 2 * Y = A + C := eval_diagonal D ...
  
  -- Step 3: Ring algebra: 2*(Y-X) = A-B = Y-X → Y-X = 0 → A = B
  have h_diff : 2 * (Y - X) - (Y - X) = 0 := by rw [...]
  have h_simp : 2 * (Y - X) - (Y - X) = Y - X := by ring
  rw [h_simp] at h_diff
```

## Experiment-Driven Discovery Path

1. **ProjectionSearch** — found that A/B/C/D classification depends only on commuting source condition
2. **ChainSearch** — found all nonadjacent coefficients reduce to adjacent sources
3. **ReductionTree** — formalized the width-decreasing DAG structure
4. **Evaluation** — proved confluence and edge-weight formulas
5. **Leaf/Adjacent** — proved skip_two_eq as first leaf relation

## Current State

- 42 theorems across 16 files
- lake build: 2966 jobs
- Pure DAG — no circular imports between layers
- Reduction→Evaluation→Leaf pipeline fully formalized for HalfDerivation
- `all_adjacent_diag_equal` (the final leaf constant theorem) is the remaining gap
