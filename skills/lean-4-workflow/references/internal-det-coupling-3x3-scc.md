# internal_det_coupling 3×3 SCC (2026-06-27)

## Context

`internal_det_coupling_zero` (ImageContainment.lean) — proves `coeff(i,i+1; u,i+1) = 0`
for `u ≥ 3`, `u < i`, `i+1 < n`. Originally believed to need a "descending chain" or
induction; proven to be a **local 3-equation SCC** instead.

## The SCC

Variables: `x = coeff(u-1,u; u-1,i)`, `y = coeff(i,i+1; u,i+1)`, `z = coeff(i,n; u,n)`

| Eq | Source | Equation |
|----|--------|----------|
| Eq1 | `bracketIdentity(i,i+1; u-1,u; u-1,i+1)` | `x + y = 0` |
| Eq2 | `bracketIdentity(u-1,u; i,n; u-1,n)` | `x + z = 0` |
| Eq3 | `coeffOf_cond(i,n; u,n)` at `k=i+1` | `2z = y` |

Matrix: `[[1,1,0],[1,0,1],[0,-1,2]]` → **det = -1** (characteristic-independent closure).

No IH, no chain, no hI, no width machinery. Pure local algebra.

## Methodology: systematic partner-audit-before-implementation

The process that discovered this:

1. **Audit all bracketIdentity configs** that can produce the target coefficient.
   For `coeff(u-1,u; u-1,i)`, enumerate (a,b,c,d,u_target,v_target) for all T1-T4.
   Only T1 and T4 paths survive; T2/T3 dead because commuting requires incompatible indices.

2. **Rank-audit the resulting equations.** Check whether the equations are actually
   independent. In this case, `bracketIdentity(A,B)` and `bracketIdentity(B,A)` give
   the SAME equation (just sign-flipped), so they don't contribute rank.

3. **Check the third equation source.** `coeffOf_cond` applies only when source width ≥ 2.
   In this case, the partner coefficient `z = coeff(i,n; u,n)` has source `(i,n)` with
   width `n-i ≥ 2` (when `i+1 < n`), so coeffOf_cond at `k=i+1` is valid.

4. **Compute the determinant.** If det ≠ 0, the SCC closes without any additional
   machinery. If det = 0, either find a new equation or accept that this approach fails.

5. **Only then implement in Lean.**

## Key pitfall avoided

The original hypothesis was a "descending chain":
```
coeff(u-1,u; u-1,i) + coeff(u-2,u-1; u-2,i) = 0
```
This equation **does not exist**. Sources `(u-1,u)` and `(u-2,u-1)` do not commute
(share index `u-1`). `bracket_zero` doesn't apply; `coeffOf_cond` requires width ≥ 2.

The audit caught this BEFORE any Lean was written.

## Edge cases

- `i+1 = n`: `coeffOf_cond` inapplicable (source `(i,n)` becomes adjacent).
  Separate handling via `boundary_family_right` (own blocker).
- `u = 1`: no commuting partner exists (partner `(0,1)` invalid). Separate blocker.

## Implementation note: field algebra without linarith

`linarith` and `nlinarith` do NOT work on `Field F`. Replace with explicit `calc` + `ring`:

```lean
-- Instead of: linarith
-- Use:
have h_eq : x + y = 0 := by
  have htemp : -(x + y) = 0 := by
    calc
      -(x + y) = -y - x := by ring
      _ = 0 := by simpa [sub_eq_add_neg, add_assoc] using hb
  exact neg_eq_zero.mp htemp

-- Instead of: have hz : z = 0 := by nlinarith
-- Use:
have hz : z = 0 := by
  calc
    z = (2 : F) * z - z := by ring
    _ = z - z := by rw [h_cond_simp]  -- h_cond_simp: 2*z = z
    _ = 0 := by ring
```
