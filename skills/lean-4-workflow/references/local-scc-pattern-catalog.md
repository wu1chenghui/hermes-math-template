# 2×2 SCC Pattern — bracketIdentity + coeffOf_cond

Discovered during the `internal_det_coupling u=1` and `boundary_family_right u≥4` closures (2026-06-27).

## Mechanism

When a GOAL coefficient appears in bracketIdentity via exactly ONE equation shape (all partners produce the same form `GOAL + F(c) = 0`), but the interior coefficient `F(c)` has a nonadjacent source where `coeffOf_cond` applies:

1. **bracketIdentity** → all interior coefficients equal: `F(1) = F(2) = ... = F(u-1) = -GOAL`
2. **coeffOf_cond** at a split point between c and j → `2·F(c) = F(k)` (ONLY C-term fires when target_row = source_row and k < target_col)
3. **Combine**: `2·F(c) = F(k) = F(c)` → `F(c) = 0` → `GOAL = 0`

This is a **2×2 SCC**: variables {F(c), F(k)}, equations {x = y, 2x = y}. det = 1, char-independent.

## When to look for this pattern

- GOAL appears in bracketIdentity only via one "family" of partners
- All partners give the same equation form (just different interior coefficients)
- The interior coefficient's source is NONADJACENT (so coeffOf_cond applies)
- coeffOf_cond's C-term (target_row = source_row) always fires
- All other terms (A/B/D) are dead because target_col ≠ source_col

## Examples from the project

### boundary_family_right(u≥4)
```
GOAL = coeff(n-1,n; u,n)
F(c) = coeff(c,u; c,n-1)  for c < u
```
- bracketIdentity(n-1,n; c,u; c,n) → GOAL + F(c) = 0 for ALL c<u
- coeffOf_cond(1,u; 1,n-1) at u-1 → 2·F(1) = F(u-1)
- → F(1) = F(u-1) = -GOAL, 2·F(1) = F(1) → F(1)=0

### internal_det_coupling(u=1)
```
GOAL = coeff(i,i+1; 1,i+1), i ≥ 2
X = coeff(i,i+2; 1,i+2)
Y = coeff(i,n; 1,n)
```
- half-Leibniz at (i,i+1)(i+1,i+2): GOAL = 2·X  (non-commuting, bracketSource ≠ 0)
- half-Leibniz at (i,i+1)(i+1,n): GOAL = 2·Y
- coeffOf_cond(i,n; 1,n) at i+2: 2·Y = X  (ONLY A-term: 1<i+2 ∧ n=n)
- → X = Y, 2·X = X → X = 0 → GOAL = 0

## Key audit steps

1. **Forward audit**: enumerate ALL bracketIdentity configurations (T1-T4) where GOAL appears
2. **Reverse audit**: for each partner family, check if GOAL appears as a partner's term
3. **coeffOf_cond audit**: for each interior coefficient, check which A/B/C/D terms fire
4. **Mechanical verification**: `lean_run_code` sanity check on each equation BEFORE writing the full proof

## Pitfalls

- For the last adjacent source (n-1,n), coeffOf_cond has NO split point → this pattern doesn't apply
- For `half_leibniz` (non-commuting sources), the equation is `BI = 2·BS` not `BI = 0`
- Always verify ALL four if-terms (A/B/C/D) — sometimes a term that "should be dead" actually fires due to index coincidences
- The `bracketIdentity_eq_expanded` leaves trailing `-0 + 0 - 0` terms; use `simpa` not direct assignment
- In Field F, `2x = x` → `(2-1)·x = 0` → `x = 0` (since 2-1=1≠0 in any field, char≠2 not needed)
