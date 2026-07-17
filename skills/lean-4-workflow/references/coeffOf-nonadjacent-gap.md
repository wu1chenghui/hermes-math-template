# coeffOf_nonadjacent Gap — Full Analysis (2026-06-22)

## The Problem

`coeffOf_nonadjacent` in `HalfNonadjacent.lean` claims:
```
u ≠ i ∨ v ≠ j → coeffOf D i j u v = 0
```

This is **mathematically false** for adjacent sources with target (1,n).
Counterexample: P₁(E₁₂) = E_{1n} is a valid 1/2-derivation with c_{12}^{1n} = 1 ≠ 0.

## What IS Correct

- For nonadjacent sources (j-i ≥ 2): ALL off-diagonal coefficients vanish — **proved**
- For adjacent sources (j-i = 1): off-diagonal coefficients vanish **except** at (1,n)

## Reachability of the sorry (line 156)

The `sorry` at line 156 is **reachable through induction**. The four recursive branches
(IA, IB, IIA, IIB) all create width-1 children whose targets are guaranteed to ≠ (1,n):

| Branch | Child source | Child target | = (1,n)? | Why |
|--------|-------------|-------------|----------|-----|
| IA | (i,i+1) | (u,i+1) | ❌ | i+1 < n (from h_ge2: i+1 < j ≤ n) |
| IB | (i,i+1) | (j,v) | ❌ | j > 1 (from 1 ≤ i < j) |
| IIA | (i+1,j) | (i+1,v) | ❌ | i+1 ≥ 2 |
| IIB | (i+1,j) | (u,i) | ❌ | i < n |

**Key finding**: The induction skeleton is correct. The nonadjacent proof is not
contaminated by the (1,n) counterexample. Only the theorem STATEMENT is too broad.

## ONLY/PAIR Limitations

ONLY/PAIR lemmas give coefficient relations (propagation tools), not direct vanishing:
- ONLY-A gives coeff(i,i+1; u, c) — second coord = chosen source index c, not arbitrary v
- ONLY-B gives coeff(i,i+1; d, v) — first coord = chosen source index d, not arbitrary u
- ONLY-C/D give second-source coefficients, not adjacent-source coefficients
- PAIR lemmas give equalities between adjacent-source and second-source coefficients

**They cannot directly prove `adjacent_noncentral_offdiag_zero`.** The conclusion
`coeff(i,i+1; u, v)` requires c=v or d=u, which contradicts the bracket index ordering c < d.

## Mutual Dependency at Same Endpoint

At endpoint j, width-2 source (j-2,j) and adjacent source (j-1,j) form an inseparable
mutual dependency:

```
A₂(j) → B(j)    (via coeffOf_cond_of split at j-1: RHS terms C and D)
B(j) → A₂(j)    (via center argument bracket at (j-2, j-1): needs T(E_{j-2,j}))
```

This IS a genuine mathematical coupling, not a proof organization artifact.

## Attempted Solutions

1. **Single lexicographic measure (j, j-i, v-u)**: Fails — within same endpoint,
   nonadjacent recursion needs width decreasing (smaller measure), center argument
   needs width INCREASING (also smaller measure). Opposite signs required.

2. **Outer endpoint + inner width induction**: Fails — width-2 at endpoint j needs
   width-1 at same endpoint via coeffOf_cond_of split.

3. **Lean `mutual` blocks**: Not available for `theorem` in Lean 4.

4. **Joint theorem returning pair** `(A₂(j) ∧ B(j))`: Promising but requires both
   conjuncts to reference each other — Lean's `constructor` blocks this.

## Current State

- `HalfNonadjacent.lean`: Added `adjacent_noncentral_offdiag_zero` lemma (with sorry),
  imported `ProjectionIdentity`, `Only`, `Pair`
- `coeffOf_nonadjacent` line 156: replaced bare `sorry` with `h_center` case split +
  call to boundary lemma
- `HalfClassification.lean`: Skeleton `offdiag_zero` theorem with endpoint induction,
  `width2_offdiag_zero` and `adjacent_noncenter_offdiag_zero` stubs
- Build passes (2967 jobs) with 5+ sorries
- Backup: `/workspace/backups/E_backup_20260622_083428_boundary_lemma/`
