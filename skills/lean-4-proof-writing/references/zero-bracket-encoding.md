# Zero-Bracket Encoding in Lean (`cond_zero`)

## Problem

When encoding a 1/2-derivation D of a Lie algebra in Lean via a coefficient
structure `NCoeff`, the standard `cond` field captures the identity:

`2D([E_{ik}, E_{kj}]) = [D(E_{ik}), E_{kj}] + [E_{ik}, D(E_{kj})]` (for i<k<j)

But this misses the **zero-bracket** case: when `[E_{ab}, E_{cd}] = 0`, the
identity gives a constraint too:

`0 = [D(E_{ab}), E_{cd}] + [E_{ab}, D(E_{cd})]`

## Solution: `cond_zero` field

```lean4
structure NCoeff where
  f : ℕ → ℕ → ℕ → ℕ → F
  cond : ∀ i j u v, … → ∀ k, i < k → k < j → …
  cond_zero : ∀ i j c d, 1 ≤ i → i < j → j ≤ n → 1 ≤ c → c < d → d ≤ n →
    j ≠ c → d ≠ i → ∀ u v, 1 ≤ u → u < v → v ≤ n →
    (if u < c ∧ v = d then f i j u c else 0)
    - (if u = c ∧ d < v then f i j d v else 0)
    + (if u = i ∧ j < v then f c d j v else 0)
    - (if u < i ∧ v = j then f c d u i else 0) = 0
```

## Derivation of the four terms

The identity `0 = [D(E_{ij}), E_{cd}] + [E_{ij}, D(E_{cd})]` expanded in the
basis `{E_{uv}}`:

**Term 1:** `[D(E_{ij}), E_{cd}]` via `[E_{pc}, E_{cd}] = E_{pd}` when q = c.
- p = u, q = c, output E_{u,d}. Requires u < c.
- Coefficient: f i j u c for target (u,d) if u < c.
- `(if u < c ∧ v = d then f i j u c else 0)`

**Term 2:** `[D(E_{ij}), E_{cd}]` via `-[E_{dq}, E_{cd}] = -E_{cq}` when d = p.
- p = d, q = v, output -E_{c,v}. Requires d < v.
- Coefficient: -f i j d v for target (c,v).
- `-(if u = c ∧ d < v then f i j d v else 0)`

**Term 3:** `[E_{ij}, D(E_{cd})]` via `[E_{ij}, E_{jq}] = E_{iq}` when j = p.
- p = j, q = v, output E_{i,v}. Requires j < v.
- Coefficient: f c d j v for target (i,v).
- `+(if u = i ∧ j < v then f c d j v else 0)`

**Term 4:** `[E_{ij}, D(E_{cd})]` via `-[E_{pi}, E_{ij}] = -E_{pj}` when q = i.
- p = u, q = i, output -E_{u,j}. Requires u < i.
- Coefficient: -f c d u i for target (u,j).
- `-(if u < i ∧ v = j then f c d u i else 0)`

## Usage for Type C interior (u < i < i+1 < v < n)

**Correct approach:** Choose `(c,d) = (v, v+1)` with target `(u, v+1)`.

Only Term 1 survives because `u < v ∧ (v+1) = (v+1)`:
```
cond_zero(i,i+1,v,v+1,...,u,v+1):
  Term 1: (if u < v ∧ (v+1) = (v+1) then D.f i(i+1) u v else 0) = D.f i(i+1) u v
  Term 2: (if u = v ∧ (v+1) < (v+1) then ...) = 0
  Term 3: (if u = i ∧ (i+1) < (v+1) then D.f v(v+1) (i+1) (v+1) else 0) = 0
  Term 4: (if u < i ∧ (v+1) = (i+1) then D.f v(v+1) u i else 0) = 0
```
Result: `D.f i(i+1) u v = 0`. ✅

**Requirements:** `v < n` (so `v+1 ≤ n`), `(i+1) ≠ v` (true since `i+1 < v`),
`(v+1) ≠ i` (true since `v > i+1 > i`).

## Distant-anchor technique: targets completely disjoint from source

When source `(i,i+1)` and target `(u,v)` are completely disjoint (either
`v < i` or `u > i+1`), the bracket `[E(i,i+1), E(u,v)] = 0`. Standard
`cond_zero` parameter choices all give `0 = 0` (no constraint), because
the condition `u < c` (Term 1) or `u = c` (Term 2) or `u = i` (Term 3) or
`u < i` (Term 4) all fail to fire simultaneously with the target indices.

**Solution: choose a distant (c,d) pair that anchors one end of the target.**
Instead of using `(u,v)` as the second source pair, use a far-away pair
such that *one* of the four terms picks up the coefficient via a partial
index match, and the residual terms are killed by the nonadjacent theorem.

### Blind A — target before source (v < i)

Choose `(c,d) = (v, n)` with target `(u, n)`:

- Term 1: `(if u < v ∧ n = n then D.f i(i+1) u v else 0)` = `D.f i(i+1) u v` ✓
- Term 2: `-(if u = v ∧ n < n then ...)` = 0 (u≠v, also n<n false)
- Term 3: `+(if u = i ∧ (i+1) < n then D.f v n (i+1) n else 0)` = 0 (u≠i)
- Term 4: `-(if u < i ∧ n = (i+1) then D.f v n u i else 0)`
  = 0 when i ≠ n-1 (n ≠ i+1)
  = `-D.f v n u (n-1)` when i = n-1 (n = i+1)

**Two cases:**
- **i ≠ n-1:** Direct vanishing: `D.f i(i+1) u v = 0`. ✅
- **i = n-1:** `D.f (n-1)n u v = D.f v n u (n-1)`. Source `(v,n)` is nonadjacent
  (length n-v ≥ 2 since v < n-1), target `(u,n-1) ≠ (v,n)`. By h_nonadj:
  `D.f v n u (n-1) = 0`. So `D.f (n-1)n u v = 0`. ✅

**Key insight:** `d = n` acts as a "far anchor" — Term 1 fires because
`d = n` equals the target end, and Term 4's residual (when it fires) is
always a nonadjacent-source coefficient.

### Blind B — target after source (u > i+1, i ≥ 2)

Choose `(c,d) = (i-1, u)` with target `(i-1, v)`:

- Term 1: `(if i-1 < i-1 ∧ v = u then ...)` = 0 (i-1 < i-1 false)
- Term 2: `-(if i-1 = i-1 ∧ u < v then D.f i(i+1) u v else 0)` = `-D.f i(i+1) u v` ✓
- Term 3: `+(if i-1 = i ∧ (i+1) < v then D.f (i-1) u (i+1) v else 0)` = 0 (i-1≠i)
- Term 4: `-(if i-1 < i ∧ v = (i+1) then D.f (i-1) u (i-1) i else 0)` = 0 (v≠i+1 since u > i+1)

Result: `-D.f i(i+1) u v = 0` ⇒ `D.f i(i+1) u v = 0`. ✅

**Boundary:** i = 1: the second source would be `(0,u)` which is invalid.
The i=1 case is handled separately via `typeB_one_vanish` or the
`cond_zero(1,2,1,u,1,v)` + h_nonadj approach.

### Blind B, i=1, u > 2 variant

Use `cond_zero(1,2,1,u,1,v)`:

- Term 2: `-(if 1 = 1 ∧ u < v then D.f 1 2 u v else 0)` = `-D.f 1 2 u v`
- Term 3: `+(if 1 = 1 ∧ 2 < v then D.f 1 u 2 v else 0)` = `D.f 1 u 2 v`

Result: `-D.f 1 2 u v + D.f 1 u 2 v = 0` ⇒ `D.f 1 2 u v = D.f 1 u 2 v`.
Source `(1,u)` is nonadjacent (length u-1 ≥ 2 since u > 2), target `(2,v) ≠ (1,u)`.
By h_nonadj: `D.f 1 u 2 v = 0`. So `D.f 1 2 u v = 0`. ✅

### Summary: when to use which distant-anchor technique

| Target position | Condition | (c,d) pair | Target (u',v') | Killed by |
|---|---|---|---|---|
| Before source | v < i | (v, n) | (u, n) | Term 1 + h_nonadj (i=n-1) |
| After source, i≥2 | u > i+1 | (i-1, u) | (i-1, v) | Term 2 |
| After source, i=1 | u > 2 | (1, u) | (1, v) | Term 2 + h_nonadj |
| Starts at source end, i≥2 | u = i+1 | (i-1, i+1) | (i-1, v) | Term 2 |

## Edge case: u = i+1, v > i+1 (i ≥ 2)

The bracket `[E(i,i+1), E(i+1,v)] = E(i,v) ≠ 0`, so `cond_zero` between
`(i,i+1)` and `(i+1,v)` doesn't apply. Instead, choose `(c,d) = (i-1,i+1)`:

- Term 2: `-(if i-1 = i-1 ∧ (i+1) < v then D.f i(i+1) (i+1) v else 0)` = `-D.f i(i+1) (i+1) v`
- All other terms vanish.

Result: `D.f i(i+1) (i+1) v = 0`. ✅ Only works for i ≥ 2 (need i-1 ≥ 1).

## When cond_zero can't resolve a coefficient

### Corner coefficient: `D.f i(i+1) 1 n` for u=1, v=n

`D.f i(i+1) 1 n` does NOT appear in any cond_zero equation, for any choice of
(c,d) or (u,v). Proof by exhaustive term classification:

**T1 path:** Need target (u',c) = (1,n). Requires c = n, but then c < d fails
(d > n). Or u' = 1, c = 1, but u' < c fails (1 < 1).

**T2 path:** Need target (d,v') = (1,n). Requires d = 1, but c < 1 fails.
Or c = 1, d = n, target (1,n) via (u',v') = (1,n): conditions are
u' = c (1 = 1) and d < v' (n < n) — fails because n < n is false.

**T3 path:** Requires (c,d) = (i,i+1) to get f i(i+1) in the cross-term.
Target becomes (j,v') = (i+1, v'). Need (1,n) = (i+1,v') → i+1 = 1 → impossible.

**T4 path:** Same as T3, target becomes (u', i+1). Need (1,n) = (u', i+1) →
i+1 = n → impossible since i+1 < n.

**Conclusion:** `D.f i(i+1) 1 n` requires a bracket-level argument outside
the cond_zero framework.

### Coupled corner coefficients: D.f (n-1)n 2 n and D.f 1 2 1 (n-1)

`cond_zero(n-1, n, 1, 2, 1, n)` gives `D.f (n-1)n 2 n = - D.f 1 2 1 (n-1)`.
This is a COUPLING, not a vanishing — neither coefficient is independently
forced to zero by any cond_zero equation. They form a 1-dimensional subspace
of `Hom(L/[L,L], Z(L))`.
