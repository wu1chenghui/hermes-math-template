# internal_det_coupling_zero — chain mechanism analysis (2026-06-27)

## Problem

`internal_det_coupling_zero` u≥3 branch: after the first bracketIdentity gives
```
coeff(i,i+1; u, i+1) + coeff(u-1,u; u-1,i) = 0
```
we need to close the residual `coeff(u-1,u; u-1,i)`.

## User's proposed chain (INVALID — sources don't commute)

```
coeff(u-1,u; u-1,i) + coeff(u-2,u-1; u-2,i) = 0
```

This equation does NOT come from any bracketIdentity because:
- `(u-1,u)` and `(u-2,u-1)` share index `u-1` → `[E_{u-1,u}, E_{u-2,u-1}] = -E_{u-2,u} ≠ 0`
- `bracket_zero` does NOT apply (sources must commute)
- Would need full `coeffOf_cond` (half-Leibniz), not simple T1-T4 expansion

## Systematic enumeration of bracketIdentity producing `coeff(u-1,u; u-1,i)`

The bracketIdentity expansion: `T1 - T2 + T3 - T4`
```
T1: if u_target < c ∧ v_target = d → coeff(a,b; u_target, c)
T2: if u_target = c ∧ d < v_target → coeff(a,b; d, v_target)
T3: if u_target = a ∧ b < v_target → coeff(c,d; b, v_target)
T4: if u_target < a ∧ v_target = b → coeff(c,d; u_target, a)
```

### T1 path
`bracketIdentity(u-1,u; i,d; u-1,d)` for any `d > i`:
- T1 fires → `coeff(u-1,u; u-1,i)` ✓
- T3 fires → `coeff(i,d; u,d)`
- Commuting: `u-1≠i` ✓, `u≠d` ✓ (for `d≠u`)
- Special case `d=i+1`: gives the SAME first-step equation, no new info.

### T2 path (IMPOSSIBLE)
`bracketIdentity(u-1,u; c,u-1; c,i)`:
- T2 fires → `coeff(u-1,u; u-1,i)` ✓
- But sources share index `u-1` (first source row `u-1` = second source column `u-1`) → DON'T COMMUTE.

### T4 path
`bracketIdentity(i,b; u-1,u; u-1,b)`:
- T4 fires → `coeff(u-1,u; u-1,i)` ✓
- T2 fires → `coeff(i,b; u,b)` 
- For `b=i+1`: same first-step equation again.
- For `b=n`: gives `coeff(u-1,u; u-1,i) = coeff(i,n; u,n)` ← **promising alternative** (see below)

## Promising alternative: partner `(i,n)`

```
bracketIdentity(u-1,u; i,n; u-1,n):
  T1: u-1 < i, n = n → coeff(u-1,u; u-1,i)       ← residual
  T3: u-1 = u-1, u < n → coeff(i,n; u,n)           ← partner term
```

Equation: `coeff(u-1,u; u-1,i) = coeff(i,n; u,n)`

- `(u-1,u)` and `(i,n)` commute: `u-1≠i` ✓, `u≠n` ✓
- `coeff(i,n; u,n)` can be reduced via `imageContainment_skeleton`'s nonadjacent split at `i+1`
- Caveat: target-width `n-u` > original `(i+1)-u`, so standard induction doesn't help — but could work within the skeleton's own nonadjacent dispatch if we organize by source-width.

## Key lesson

When designing chain/cluster closures, always check:
1. Do the two sources commute? (need `a≠c` AND `b≠d` for `[E_ab, E_cd] = 0`)
2. If they don't commute, bracketIdentity won't give a simple T1-T4 expansion — need full half-Leibniz/coeffOf_cond.
