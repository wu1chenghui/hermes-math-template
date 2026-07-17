# Boundary Rigidity — Complete Proof

## Problem

For a centered half-derivation D of N_n (char F ≠ 2, n ≥ i+2, i ≥ 3), prove:
```
coeff(D; i, i+1; i, n) = 0
```

This was the single LIVE sorrie blocking D3 (HalfDer = F·Id ⊕ kerΦ).

## Proof (3 equations, no I_filtered)

Define:
```
a = coeff(D; 1, i+1; 1, n)   [wide-source b-channel]
b = coeff(D; 2, i+1; 2, n)   [wide-source b-channel]
g = coeff(D; i, i+1; i, n)   [GOAL — adjacent source, target col n]
```

Three half-Leibniz equations (all valid for i ≥ 3, n ≥ i+2):
```
(1) src=(1,2), ptr=(2,i+1), tgt=(1,n) → 2a - b = 0
(2) src=(1,i), ptr=(i,i+1), tgt=(1,n) → 2a - g = 0
(3) src=(2,i), ptr=(i,i+1), tgt=(2,n) → 2b - g = 0
```

In each case, bracketSource = the wide-source coefficient (2× weight),
and only T3 survives in bracketIdentity (T1 dead because n ≠ i+1,
T2 and T4 dead because u ≠ c and u ≮ i respectively).

### Synthesis

From (1): b = 2a
From (2): g = 2a
From (3): g = 2b = 2·(2a) = 4a

Thus 4a = 2a → 2a = 0 → a = 0 (char ≠ 2) → g = 0.

## Dependencies

- Three half-Leibniz equations (bracketIdentity, bracketSource)
- CharNeTwo (2 ≠ 0)
- Centered condition (for the n=i+1 edge case where T1 fires at a diagonal)

**NO I_filtered, NO kerΦ, NO width_b_zero, NO coeffOf_cond, NO Spanning.**

## Lean formalization

File: `E/Classification/BoundaryRigidity.lean`
- `boundary_rigidity_eq1/2/3`: bracketIdentity = single coefficient
- `boundary_rigidity_bracketSource_*`: bracketSource expansions
- `boundary_rigidity_relation`: g = 2a (any half-derivation)
- `boundary_rigidity`: g = 0 (centered D)

All theorems parameterized: `(i : ℕ) (hi : 3 ≤ i) (hn : i + 2 ≤ n)`.
Build: `lake build E.Classification.BoundaryRigidity` (2953 jobs, 0 errors).

## Discovery methodology

1. **Computational experiment**: Built full half-Leibniz + centered linear system
   for n=5,6,7. Computed nullspace dimension = n+4, confirming the theorem is true.
2. **Syzygy extraction**: Traced RREF pivot dependencies. Found that the GOAL
   coefficient is isolated by just 3 equations out of 450+.
3. **Parameterized proof**: Generalized the 3-equation pattern to arbitrary i,n.
4. **Lean formalization**: Wrote BoundaryRigidity.lean with split_ifs + omega.

## Why the original ImageContainment approach failed

The original approach tried to prove `coeff(i,i+1;i,n) = 0` via:
- Commuting-partner bracketIdentity (occurrence audit showed GOAL structurally invisible)
- bracketSource → diagonal coupling (degenerate: all terms become 0=0)
- width_b_zero (requires I_filtered, which is circular)

The 3-equation syzygy was invisible to these approaches because:
- Eq (1) and (3) involve sources (1,2) and (2,i) which are NOT the adjacent source (i,i+1)
- The cancellation happens across equations, not within a single one
- Each individual equation looks like "2a - g = 0" which by itself doesn't force 0
