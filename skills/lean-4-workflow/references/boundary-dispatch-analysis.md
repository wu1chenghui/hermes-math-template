# Boundary Dispatch Analysis — BND-L / BND-R Structure

## BND-L 3×3 Local SCC (boundary_local_234)

Discovery (2026-06-26): the BND-L residual `coeff(3,n;2,n)` can be closed by a purely
local 3×3 linear system — NO hI, NO induction, NO width machinery.

```
x := coeff(3,n;2,n)     y := coeff(3,4;2,4)     z := coeff(1,2;1,3)

(1) bracketIdentity(1,2,3,n,1,n) = 0  →  z + x = 0   [T1: 1<3,n=n; T3: 1=1,2<n]
(2) bracketIdentity(3,4,1,2,1,4) = 0  →  -y - z = 0   [T2: 1=1,2<4; T4: 1<3,4=4]
(3) coeffOf_cond(3,n;2,n) @ k=4       →  2x = y       [sole A-term]
→ z=-x, y=-z=x, 2x=x → x=0 → y=z=0.  det=1, char-independent.
```

All brackets use COMMUTING partners → `bracket_zero` → 0.  No bracketSource needed.

Implementation: `ImageContainment.lean`, ~80 lines.  Signature:
```lean
lemma boundary_local_234 (D : HalfDerivation F n) (hn5 : 5 ≤ n) :
    coeffOf D 1 2 1 3 = 0 ∧ coeffOf D 3 4 2 4 = 0 ∧ coeffOf D 3 n 2 n = 0
```

## BND-L Family (boundary_family)

For all v with (1,v) ∉ I (v ≥ 3, v ≤ n-2):

```
bracketIdentity(1,2, v,v+1, 1,v+1) = 0
T1: 1<v, v+1=v+1 → coeff(1,2;1,v)    ← THE GOAL
T3: 1=1, 2<v+1   → coeff(v,v+1;2,v+1) ← interior, closed by internal_det_step
→ coeff(1,2;1,v) = -coeff(v,v+1;2,v+1) = 0  (for v≥4)
```

v=3 handled by boundary_local_234.1 directly.

Key: T1 fires because target row 1 < partner source row v (for v≥3).

## BND-R Analysis — Why the Dispatch Slot is Unreachable

In `imageContainment_skeleton`, the BND-R dispatch sits at `u=i ∧ i+1=n`:
- Source: (n-1, n)
- Target: (u, n) with u = i = n-1
- → Target = Source = diagonal → excluded by `h_off_diag`
The BND-R dispatch slot is unreachable.  The true right-boundary cases (source (n-1,n)
with u < n-1) belong in the `u < i` dispatch branch, not `u = i`.

## D fires in width_a: two cases, one unreachable

D fires appears in TWO places in `width_a_chain_zero`:

### D fires 1 — width-2 case (k=n-2, j'=n): REACHABLE
Source (n-2, n), target (1, n-1).  coeffOf_cond at k+1=n-1:
D-term = coeff(n-1,n; 1, n-2).  Closed by `boundary_coupling_sigma.2`
(σ-mirror of `boundary_coupling`).

### D fires 2 — wider case (j' > k+2): UNREACHABLE
Requires k=n-2 (so k+2=n) AND j' > k+2 = n AND j' ≤ n.
Contradiction: n < j' ≤ n.  **Path is unreachable** (`exfalso; omega`).
with u < n-1) belong in the `u < i` dispatch branch, not `u = i`.

## bracketIdentity Partner Selection — "Free Coefficient" Detection

When analyzing an adjacent source (i,i+1) with target (u,v), check WHICH bracketIdentity
terms can fire for which partners:

**T1**: `u < c ∧ v = d` → coeff(i,i+1; u,c).  Fires when target-col = partner-col (d=v)
AND target-row < partner-row (u<c).  Gives target-col = c (NOT v unless c=v in which
case partner is (v,d) with d>v).  For this to give the goal (u,v): need c=v and d=v.
But c<d → (v,v) invalid.  **T1 NEVER directly gives the goal when v is the target col.**

**T2**: `u = c ∧ d < v` → coeff(i,i+1; d,v).  Fires when target-row = partner-row (u=c)
AND partner-col < target-col (d<v).  For goal (u,v): need d=u.  But d>c=u, so d>u ≠ u.
**T2 NEVER directly gives the goal.**

**T3**: `u = i ∧ j < v` → coeff(c,d; j,v).  Fires on the PARTNER source, not the goal.

**T4**: `u < i ∧ v = j` → coeff(c,d; u,i).  Fires on the partner source.

**Conclusion**: For source (i,i+1) with target (u,v), T1 and T2 NEVER give the goal
coefficient.  The bracketIdentity constrains PARTNER coefficients (via T3/T4), not
the source's own coefficients.

**"Free coefficient" case**: when T3 and T4 are also false (u ≠ i and v ≠ j, i.e.,
u ≠ i and v ≠ i+1), bracketIdentity is TRIVIALLY ZERO for all partners.  The
coefficient is NOT constrained by bracketIdentity.  The constraint comes from
bracketSource through the half-Leibniz equation:
```
2·bracketSource(i,i+1, c,d, u,v) = bracketIdentity(i,i+1, c,d, u,v) = 0
→ 2·(coeff(i,d;u,v) - coeff(c,i+1;u,v)) = 0
```
This couples the adjacent coefficient to NONADJACENT coefficients, which require
the induction hypothesis or width machinery.

**Examples of "free" patterns**:
- (i,i+1) → (i+2, n) with i ≤ n-3: T1(u<c∧n=d) → d=n, c>u, T1=coeff(i,i+1;i+2,c) ≠ goal
- (i,i+1) → (u, v) with u<i, v≠i+1: T4(u<i∧v=i+1) → v≠i+1, false. T1/T2 can't give goal.

## Design Principles (from user feedback)

1. **验证结构命题 before proof**: when discovering a new local mechanism, FIRST verify
   the structural shape (which bracketIdentity terms fire for which partner), THEN write
   the proof.  Don't write "arbitrary v>3" until the uniformity is confirmed.
2. **Don't replace the entire dispatch at once**: add a new independent lemma, verify it
   compiles, THEN refactor the call site.  Keep old proofs archived (e.g. `_old` suffix).
3. **"Free coefficient" → induction, not bracket hack**: when bracketIdentity is trivially
   zero for all partners, the coefficient belongs in the induction/nonadjacent machinery.
   Don't try to force a bracketIdentity proof.
