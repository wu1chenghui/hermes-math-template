# A-Fire SCC Mechanism (D3 ImageContainment)

## Problem

In `width_a_chain_zero` (σ-mirror of `width_c_chain_zero`), the A-fire case
produces child `coeff(k,k+1; 1,k+1)` where k+1 < n-1 (target not in I).

This child is NOT the σ-mirror of `internal_det_step` (which targets `(2,i+1)`).
It forms a **3×3 SCC** with characteristic-independent determinant = 2.

## Variables (3 nodes)

```
A = coeff(k,  k+1; 1, k+1)   ← goal
B = coeff(k,  k+2; 1, k+2)
C = coeff(k,  k+3; 1, k+3)
```

Requires k ≤ n-4 (so k+3 ≤ n-1, source (k,k+3) valid).

## Equations (3 edges)

| Eq | Type | Equation | Mechanism |
|----|------|----------|-----------|
| 1  | half-Leibniz | A = 2·B | partner (k+1,k+2), bracket=E_{k,k+2}, T1=A |
| 2  | half-Leibniz | B = 2·C | partner (k+2,k+3), bracket=E_{k,k+3}, T1=B |
| 3  | coeffOf_cond | A = 2·C | source (k,k+3), split k+1, only A-term fires |

## Resolution

```
A = 2·B = 4·C    (Eq1+Eq2)
A = 2·C          (Eq3)
→ 4·C = 2·C
→ 2·C = 0
→ C = 0          (char≠2)
→ A = 0
```

det = 2, completely characteristic-independent (only needs char≠2).

## Anti-pattern: exponential chain

The `diagonal_shift` lemma gives `coeff(k,d;1,d) = 2·coeff(k,d+1;1,d+1)`.
Iterating this creates an exponential chain leading to
`(2^{n-k-2} - 2)·c = 0`, which is characteristic-dependent:
- char 3: fails when n-k-2 ≥ 3
- char 5: fails when n-k-2 ≥ 5
- etc.

**Do NOT use the exponential chain for A-fire closure.** The 3×3 SCC is
the correct local mechanism. `diagonal_shift` is retained as a standalone
lemma but NOT used as the primary closure tool.

## Edge case

k = n-3 (width-2 to n-1): no valid k+3 node. Handled by dispatch or
separate argument. This is structurally different from the A-fire SCC.

## Contrast with internal_det_step

| Property | internal_det_step | a_fire_scc_close |
|----------|-------------------|------------------|
| Target | (2, i+1) | (1, k+1) |
| Structure | 3-eq 2×2 | 3-eq 3×3 |
| det | 1 | 2 |
| IH needed | No | No |
| Mechanism | bracket_zero + bracket_zero + coeffOf_cond | half-Leibniz + half-Leibniz + coeffOf_cond |
| Partner | (1,2) commuting | (k+1,k+2) and (k+2,k+3) non-commuting |

## Lean implementation

See `ImageContainment.lean`:
- `diagonal_shift`: single-step lemma (local bracketIdentity)
- `a_fire_scc_close`: full 3×3 SCC closure (0 sorry)

Key pitfalls:
- `linarith` does NOT work on generic `Field F` — use `ring` + manual algebra
- `coeffOf_f` rewrites `D.coeff ↔ D.coeffOf`; use `simpa [coeffOf_f ...]` not `rw`
- `half_leibniz` expects target validity `hu : 1 ≤ u` for row 1, use `by omega`
