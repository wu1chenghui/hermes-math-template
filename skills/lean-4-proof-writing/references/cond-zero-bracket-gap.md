# coeffOf_cond_zero: bracket expansion identity (gap)

## Mathematical statement

When `[E(i,j), E(c,d)] = 0` (i.e., `j ≠ c` and `d ≠ i`), the half-derivation
identity `2·D([x,y]) = [D(x),y] + [x, D(y)]` specialises to:

```
0 = [D(E(i,j)), E(c,d)] + [E(i,j), D(E(c,d))]
```

Expanding both brackets in the basis `{E(u,v)}` gives the 4-term `cond_zero` equation:

```
(if u<c ∧ v=d then f(i,j)(u,c) else 0)
- (if u=c ∧ d<v then f(i,j)(d,v) else 0)
+ (if u=i ∧ j<v then f(c,d)(j,v) else 0)
- (if u<i ∧ v=j then f(c,d)(u,i) else 0)
= 0
```

## Why it's hard

The `coeffOf_cond` lemma connects `half_deriv_cond` (splitting a single source
at `k`) to `NCoeff.cond`.  But `coeffOf_cond_zero` is a DIFFERENT identity —
it's about bracket expansion of TWO commuting sources, not splitting one source.

The `half_deriv_cond` gives `2·D([E(i,k), E(k,j)]) = [D(E(i,k)), E(k,j)] + [E(i,k), D(E(k,j))]`
which relates D(E(i,j)) to D(E(i,k)) and D(E(k,j)).  The `cond_zero` identity
gives `[D(E(i,j)), E(c,d)] + [E(i,j), D(E(c,d))] = 0` which is about the
bracket interaction of two DIFFERENT half-derivations.

Neither identity implies the other directly — both follow from the general
half-derivation property `2·D([x,y]) = [D(x),y] + [x, D(y)]` combined with
linearity of D.

## Required infrastructure

To prove `coeffOf_cond_zero`, one needs:

1. **Bracket expansion in the coefficient basis**: for any `f(a,b)(p,q)` = coefficient
   of E(p,q) in D(E(a,b)), the coefficient of E(u,v) in `[D(E(i,j)), E(c,d)]` is:

   ```
   (if v=d ∧ u<c then f(i,j)(u,c) else 0) - (if u=c ∧ d<v then f(i,j)(d,v) else 0)
   ```

   This follows from the combinatorial rule `[E(p,q), E(c,d)] = E(p,d)` if q=c,
   `= -E(c,q)` if d=p, `= 0` otherwise.

2. **Linearity of D**: D is F-linear. While `HalfDerivation` doesn't explicitly
   encode linearity, it's provable from `half_deriv_cond` for nilpotent Lie algebras.
   Without linearity, the bracket of a SUM (D(E(i,j)) = Σ coeffOf D i j p q · E(p,q))
   cannot be decomposed term-by-term.

3. **Half-derivation property for zero bracket**: for `[x,y] = 0`, we have
   `[D(x),y] + [x, D(y)] = 2·D([x,y]) = 2·D(0) = 0`. This requires D(0) = 0,
   which follows from linearity or from `coeffOf` returning 0 for invalid indices.

## Proof approach (conceptual)

1. Define `bracketCoeff (a b p q u v : ℕ) : F` = coefficient of E(u,v) in [E(a,b), E(p,q)]
   (returns ±1 or 0 depending on index matches)
2. Prove `bracketCoeff` expansion lemma (case analysis on `b=p`, `q=a`)
3. For `D(E(i,j)) = Σ coeffOf D i j p q · E(p,q)`, expand `[D(E(i,j)), E(c,d)]`
   as `Σ coeffOf D i j p q · bracketCoeff p q c d u v`
4. Show the sum reduces to the 2-term expression (q=c or d=p contributions)
5. Similarly for `[E(i,j), D(E(c,d))]`
6. Sum both brackets, use `half_deriv_cond` on the zero bracket to get 0

## Current status

**Tracked in**: `E/HalfDerivation/ToNCoeff.lean` — `coeffOf_cond_zero` with FIXME.
**Unblocks**: `adjacent_reduce_to_corner` (NCoeff level) → `ImageInCenter` → `dim ≤ n`.
**Priority**: Medium — the main classification chain works if this is accepted as an axiom.

## ⚠️ Anti-pattern (do NOT use)

The current Bracket.lean proves cases by calling `coeffOf_nonadjacent` to kill
individual surviving coefficients after `split_ifs`. This is WRONG because:

1. `coeffOf_nonadjacent` has its own adjacent-source gap
2. It creates a circular dependency: Bracket → ToNCoeff → Adjacent → Classification → Bracket
3. It treats the problem as "kill each coefficient" instead of "the bracket identity
   forces the sum to zero"

## 方案 C (recommended approach)

See `references/coeffOf-cond-zero-strategy-C.md` for the full strategy.

Core insight: prove the 4-term identity as a **local combinatorial bracket expansion**,
not by proving each coefficient individually zero. The bracket identity `[D(E_ij), E_cd] + [E_ij, D(E_cd)] = 0`
already gives A - B + C - D = 0 directly — no `coeffOf_nonadjacent` needed.

## Workaround

Until `coeffOf_cond_zero` is proven, the `toNCoeff D : NCoeff F n` can be used
for lemmas that only need `cond` (e.g., `NCoeff.diagonalization`, `T_nonadjacent_zero`).
The `cond_zero`-dependent lemmas (`adjacent_reduce_to_corner`, `T_adjacent_support`)
remain `sorry`.
