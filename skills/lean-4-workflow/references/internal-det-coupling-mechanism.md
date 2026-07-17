# Internal-Det Coupling — Mechanism Investigation (2026-06-26)

## Problem

`internal_det_coupling_zero` in `ImageContainment.lean` should prove:
```
coeffOf D i (i+1) u (i+1) = 0    where u < i, (u,i+1) ∉ I, i+1 < n
```

The lemma is called from `imageContainment_skeleton` dispatch (line 741, interior u<i case).

## What does NOT work: partner (i,n) + coeffOf_cond

The seemingly-elegant approach:
1. `bracketIdentity(i,i+1, i,n, u,i+1) = 0` → T4 fires → `coeff(i,n; u,i) = 0`
2. `coeffOf_cond_of D i n u i` at split `m = i+1` → only A fires → relates to goal

**WHY IT FAILS:** `coeffOf_cond_of`'s A-term guard is `u < m ∧ target_column = source_column`:
- Here: `u < i+1 ∧ i = n`
- Since `i+1 < n` (guaranteed by dispatch — BND-R catches `i+1=n`), `i ≠ n`
- Guard is FALSE → A-term = 0, not our goal

The bracketIdentity gives us `coeff(i,n; u,i)` — target column `i` equals source row `i`.
This creates a "dead-end" coefficient where coeffOf_cond's guards all fail because
the target column (=source row) matches none of the source or split columns.

## What DOES work (partial understanding)

For the `u=2` subcase (which covers all width_c_chain sorries):

Partner `(c,u)` for any `c < u`:
```
bracketIdentity(i,i+1, c,u, u,i+1):
  T2 fires → coeff(i,i+1; u,i+1) = -coeff(c,u; u,i)
```
When `u=2`, partner `(1,2)` gives:
```
coeff(i,i+1; 2,i+1) = -coeff(1,2; 2,i)    [BND-L chain]
```
This reduces to the BND-L cluster — which is what `width_c_chain_zero` handles.

## Implementation-order rule

**Do NOT fill `width_c_chain_zero` sorries that need `internal_det_coupling_zero` results**
before implementing `internal_det_coupling_zero` itself. The four remaining sorries in
`width_c_chain_zero` (A-fires, C-fires, width-2 k=2, A+C both) all depend on interior
det-coupling results. Building them first creates circular proof attempts.

The correct order per AGENTS.md Step C:
1. `internal_det_coupling_zero` (interior det cluster)
2. `width_a_chain_zero` (σ-mirror)
3. `width_c_chain_zero` remaining sorries (call both of the above as APIs)
4. `imageContainment` assembly

## Dispatch guarantee

`internal_det_coupling_zero` is only called when `i+1 < n` (BND-R dispatch at line 726-728
catches `i+1 = n` first). The lemma's proof can assume this without proving it.
