# 方案 C: coeffOf_cond_zero as a local combinatorial identity

## The strategic insight

`coeffOf_cond_zero` is NOT a coefficient-vanishing theorem. It is a **bracket expansion
combinatorial identity**. The proof should establish the 4-term sum identity directly
through bracket decomposition, never through `coeffOf_nonadjacent` (which creates a
circular dependency with Adjacent/Classification).

## Anti-pattern (the current failed approach)

```
split_ifs
    ↓
surviving term: coeffOf D i j u c
    ↓
call coeffOf_nonadjacent  ← circular dependency
    ↓
coeffOf D i j u c = 0
    ↓
identity holds
```

This is wrong because:
- `coeffOf_nonadjacent` has an adjacent-source gap (same class of problem)
- It creates a cycle: Bracket → ToNCoeff → Adjacent → Classification → Bracket
- It treats each branch as "kill the surviving coefficient" rather than "the identity
  forces the surviving coefficient to be zero"

## 方案 C (correct approach)

The 4-term identity `A - B + C - D = 0` comes from:

```
[E_ij, E_cd] = 0  (j≠c, d≠i)
    ↓ half-derivation property: 2·D([x,y]) = [D(x), y] + [x, D(y)]
[D(E_ij), E_cd] + [E_ij, D(E_cd)] = 0
    ↓ expand each bracket to the E_uv basis
A - B + C - D = 0
```

For each split_ifs case, the surviving terms are identified by **which bracket
decompositions can actually produce E_uv**, not by which coefficients happen to be
nonzero. This is a support analysis, not a coefficient analysis.

### Example: "Only A" case

Rather than proving `coeffOf D i j u c = 0` by `coeffOf_nonadjacent`:

Show that in the bracket expansion, the A term is the **only** term that can
produce E_uv when `u<c ∧ v=d` and all other Kronecker conditions fail. Since the
sum is zero (by half-derivation property), the sole contributor must be zero.

This requires no knowledge of whether (i,j) or (c,d) is adjacent — it's purely
about the bracket combinatorics.

### Example: "A+C" pair case

Rather than proving A=0 and C=0 separately:

Show that A and C are the **only two** terms that can produce E_uv. Then A+C=0
follows directly from the half-derivation identity. No individual coefficient
needs to be proven zero.

## Implementation paths

### Path 1: Bracket expansion bridge (recommended for cleanliness)

Infrastructure needed:
1. Define `HalfDerivation.toLinearMap` — lift `coeff` to a linear map `𝔫 → 𝔫`
   where `𝔫 := MatIdx n → F` (already defined in HalfDerivation.lean)
2. Define bracket on `𝔫` as a bilinear operation (lift `bracket : MatIdx n → MatIdx n → Option (ℤ × MatIdx n)`)
3. Prove `[D(E_ij), E_cd]` expands to `Σ coeffOf D i j p q · [E_pq, E_cd]` (linearity)
4. Prove `[E_pq, E_cd]` reduces to at most ONE basis element (Kronecker δ)
5. Read off the coefficient of `E_uv`: exactly Term A and Term B
6. Similarly for `[E_ij, D(E_cd)]`: Term C and Term D
7. Sum = 0 by half-derivation property

Estimated: ~150-250 lines of new infrastructure, then `coeffOf_cond_zero` becomes ~10 lines.

### Path 2: coeffOf_cond bidirectional (lighter, requires more case cleverness)

Without building the full bracket expansion, use `coeffOf_cond` to relate the
surviving `coeffOf D i j ...` term to a `coeffOf D c d ...` term, forming a
pair identity directly. This avoids the linear-map infrastructure but requires
more per-case reasoning.

## Key design principle

> **`Bracket.lean` should be a library of local combinatorial identities,**
> **not a mini-Classification.lean.**

It should ONLY depend on:
- `Infrastructure.lean` (MatIdx, bracket, HalfDerivation)
- `coeffOf_cond` (half-derivation splitting, already proven in Bracket.lean)

It should NOT depend on:
- `coeffOf_nonadjacent` (creates the cycle)
- `Diagonalization.lean`, `Adjacent.lean`, or anything downstream

## Relation to project architecture

Once 方案 C is implemented, the dependency chain becomes a strict DAG:

```
Bracket (pure combinatorial identities)
    ↓
ToNCoeff (trivial bridge)
    ↓
Adjacent (NCoeff analysis)
    ↓
ImageInCenter
    ↓
Classification (dim_le_n, classification: pure wrapping)
```

No cycles. No adjacent-source knowledge leaks into the combinatorial layer.
