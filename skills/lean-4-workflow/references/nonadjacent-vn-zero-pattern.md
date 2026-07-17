# Nonadjacent v=n Zero Pattern

For source `(i,j)` with `j-i ≥ 2` and target column `n` outside I:
prove `coeffOf D i j u n = 0` via decomposition at `k = i+1`.

## 4-case dispatch (coeffOf_source_exists)

| Case | Child source | Child target | Closure method |
|------|-------------|-------------|---------------|
| IA | `(i,k)` | `(u,k)`, `k<n` | `offdiag_zero_v_lt_n` (v<n satisfied) |
| IB | `(i,k)` | `(j,n)`, `j<n` | **adjacent_above_zero** (child source is `(i,i+1)` — adjacent!) |
| IIA | `(k,j)` | `(k,n)` | Recursion or BoundaryRigidity (source-width decreases) |
| IIB | `(k,j)` | `(u,i)`, `i<n` | `offdiag_zero_v_lt_n` (v<n satisfied) |

## Key insight for IB

The IB child has ADJACENT source `(i, i+1)` because `k = i+1`.
This means NO induction/recursion is needed — the existing adjacent helper
(`adjacent_above_zero` for `u > i+1`) closes it directly.

For `u = i+1` child: `adjacent_rowcol_zero`.
For `u < i` child: `adjacent_below_zero`.

This is why `nonadjacent_target_n_zero` can be Phi-only with no centeredness.

## IIA analysis

| Subcase | Condition | Closure |
|---------|-----------|---------|
| j-i ≥ 3, j≠n | Recursion via `nonadjacent_target_n_zero` | Source width ↓ |
| j-i ≥ 3, j=n | `h_off_diag` contradiction | `u=i` → `j≠n` forced |
| j-i = 2, n ≥ i+3 | `boundary_rigidity (i+1)` | Adjacent child, u=source_row |
| j-i = 2, n = i+2 | `h_off_diag` contradiction | `j=i+2=n` contradicts `j≠n` |

## Lemma signature (Phi-only, no centeredness)

```lean
lemma nonadjacent_target_n_zero (D : HalfDerivation F n) (hn3 hn5 : ...) (h3 : (3:F)≠0)
    (i j u : ℕ) (hi hij hjn hu hu_n : ...)
    (h_off_diag : i ≠ u ∨ j ≠ n) (h_notI : (u,n) ∉ I_target_set hn3)
    (h_ge2 : j-i ≥ 2) : coeffOf D i j u n = 0
```

This lemma is PURE Phi-only — no I_filtered, no kerΦ, no centeredness.
It fits in the same layer as BoundaryRigidity, adjacent_above_zero, etc.
