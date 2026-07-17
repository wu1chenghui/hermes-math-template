# Single-Bracket-Zero σ-Mirror Pattern

For proving `coeff(i,i+1; target) = 0` where the target is an I-ideal generator
and the sources commute, a SINGLE `bracket_zero` call suffices — no 3-equation
2×2 system needed.

## Pattern: `internal_det_step_a` (target `(1,n-1)`)

**Goal**: `coeffOf D i (i+1) 1 (n-1) = 0` for `2 ≤ i ≤ n-3`.

**Proof (one `bracket_zero` call)**:
```lean4
-- Sources (i,i+1) and (n-1,n) commute when i+1 ≠ n-1 ∧ i ≠ n.
-- bracketIdentity(i,i+1; n-1,n; 1,n) = 0 via bracket_zero.
-- Expansion: T1 = coeff(i,i+1;1,n-1)  [fires: 1 < n-1 ∧ n = n]
--            T3 = 0                   [fires only at i=1]
--            T4 = 0                   [fires only at n=i+1]
D.bracket_zero i (i+1) (n-1) n 1 n ...
```

**Why this works**: `[E_{i,i+1}, E_{n-1,n}] = 0` when `i+1 ≠ n-1` and `i ≠ n`.
For `2 ≤ i ≤ n-3`, both hold. The bracketIdentity at target (1,n) expands to
exactly one term: T1 = `coeff(i,i+1;1,n-1)`. All other terms have false guards.

**Contrast with `internal_det_step`** (target `(2,i+1)`):
The c-channel version requires THREE equations (two bracketIdentity + one
coeffOf_cond) forming a 2×2 linear system with `det=1`. The a-channel version
is dramatically simpler because target `(1,n-1)` is itself an I-generator,
so a bracket with the other I-pair `(n-1,n)` at the center target `(1,n)`
directly exposes the goal coefficient.

## Width-a chain mirroring

`width_a_chain_zero` mirrors `width_c_chain_zero` exactly:
- Same induction: `Nat.decreasingInduction` on source row
- Same split: `coeffOf_cond_of` at `m = k+1`
- Target change: `(2,n)` → `(1,n-1)`

Child dispatch differences (`coeffOf_cond_of` expansion terms `A-B+C-D`):

| Term | c-chain guard | a-chain guard | When fires |
|------|--------------|--------------|------------|
| A | `2<m ∧ n=j'` | `1<m ∧ n-1=j'` | `j'=n-1`, `k≥1` |
| B | `2=m ∧ j'<n` | `1=m ∧ j'<n-1` | **NEVER** (m=k+1≥2) |
| C | `2=k ∧ m<n` | `1=k ∧ m<n-1` | `k=1` only |
| D | `2<k ∧ n=m` | `1<k ∧ n-1=m` | `k=n-2` only |

Key simplification: B-term NEVER fires in the a-chain (since `m=k+1≥2`,
`1=m` is impossible). In the c-chain, B fires at `k=1` (the boundary leaf).

## Boundary cases

For `2 ≤ i ≤ n-3`: the single bracket_zero closes the proof directly.

For `i=1`: T3 fires alongside T1, giving the coupling equation
`coeff(1,2;1,n-1) + coeff(n-1,n;2,n) = 0`. Needs a second equation to
close — the width_a chain's C-fire handles this separately.

For `i=n-2`: sources DON'T commute (`i+1=n-1`). The bracket produces
`E_{n-2,n}`. This is the D-fire case in width_a_chain, handled separately.

For `i=n-1`: T1 and T4 both fire giving `coeff(n-1,n;1,n-1) - coeff(n-1,n;1,n-1) = 0`
(trivial identity, no information about the coefficient). Boundary case.
