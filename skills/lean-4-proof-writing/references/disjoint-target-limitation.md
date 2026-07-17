# Disjoint-Target Limitation in NCoeff

## The Problem

For an adjacent source (i,i+1) and a completely disjoint target (u,v) where
- v < i (target entirely before source), or
- u > i+1 (target entirely after source),

the NCoeff `cond` and `cond_zero` axioms do not constrain `D.f i(i+1) u v`.
All four terms in `cond_zero` evaluate to zero because the index sets are
disjoint.

## Why It Happens

**`cond`** requires a splitting index `k` with `i < k < i+1`. For adjacent
sources (j = i+1), no such `k` exists — this is the fundamental "adjacent
source gap."

**`cond_zero`** has four terms, each requiring the target `(u,v)` to overlap
with the source `(i,i+1)` in one of four ways:

| Term | Condition for non-zero | Why it fails for disjoint targets |
|------|----------------------|-----------------------------------|
| T1   | `u < c ∧ v = d`      | Need `v = d` — sets d=v, but then T1 gives `f i j u c` not `f i j u v` |
| T2   | `u = c ∧ d < v`      | Gives `f i j d v` — different coefficient, or `u = c` fails for c = u |
| T3   | `u = i ∧ j < v`      | `u = i` fails when u < i; `j = i+1 < v` fails when v ≤ i+1 |
| T4   | `u < i ∧ v = j`      | `v = j = i+1` fails when v < i or v > i+1 |

All four conditions require the target to share at least one index with
the source. For completely disjoint targets, none fire.

## What Is Constrained

The NCoeff axioms constrain coefficients for targets that OVERLAP with
the source:

| Overlap pattern | Constrained by | Covers |
|----------------|---------------|--------|
| v = i+1 (same end) | cond via source (i,i+2) | Type A (u < i) |
| u = i (same start) | cond via source (i-1,i+1) | Type B (v > i+1, i ≥ 2) |
| u < i < i+1 < v (straddle) | cond_zero(i,i+1,v,v+1,u,v+1) | Type C interior (v < n) |
| v = n, u > 1 (boundary) | cond_zero(i,i+1,1,u,1,n) | Type C boundary (i+1 < n) |
| u=1, i=1, 2<v, v+1<n | cond_zero(1,2,v,v+1,1,v+1) + Type A | Type B(1,v) |
| i=n-1, u=2, v=n | cond_zero(n-1,n,1,2,1,n) | Corner coupling |

## Impact on `adjacent_reduce_to_corner`

The theorem `adjacent_reduce_to_corner` (for adjacent source E(i,i+1),
all off-diagonal non-corner coefficients are zero) cannot be proven from
NCoeff axioms alone for disjoint targets. Four specific cases remain:

1. v < i — target before source
2. u > i+1 — target after source
3. i=n-1, v=n, u>2 (u≠2) — non-corner boundary
4. i=1, u=1, v=n-1 — Type B(1,n-1) boundary (coupled with case 3 by corner_relation)

## Resolution Paths

1. **Bracket-level proof**: The actual `HalfDerivation` structure has
   the full half-derivation condition `2·T([x,y]) = [T(x),y] + [x,T(y)]`,
   which for disjoint sources `[E(i,i+1), E(u,v)] = 0` gives a relation
   involving coefficients of OTHER disjoint sources. May require a
   global induction.

2. **Extra NCoeff axiom**: Add a `cond_disjoint` axiom to `NCoeff` that
   encodes the bracket-level constraint for commuting pairs, using a
   different decomposition than the existing `cond_zero`.

3. **Dimension argument**: If the total dimension of Der_{1/2}(N_n) is n
   (proved by other means), then the disjoint-target coefficients must
   be zero for dimension reasons.
