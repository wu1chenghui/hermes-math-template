# Semantics Bridge — Operator-level bracket identity

## Problem

`coeffOf_cond_zero` in `Bracket.lean` proves `A−B+C−D = 0` via a 16-case `split_ifs`.
The non-adjacent branches call `coeffOf_nonadjacent`, which (in its current form)
has an adjacent-source gap. This creates a circular dependency:

```
Bracket.lean (coeffOf_cond_zero)
    ↓ needs
coeffOf_nonadjacent (adjacent gap)
    ↓ needs
Adjacent.lean (adjacent classification)
    ↓ needs
ToNCoeff.lean
    ↑ depends on
Bracket.lean (coeffOf_cond_zero)  ← cycle
```

## Solution: Semantics Bridge

Instead of proving individual coefficients vanish (`coeffOf_nonadjacent`),
prove the bracket identity at the **operator level**:

```
[D(E_ij), E_cd] + [E_ij, D(E_cd)] = 0   when [E_ij, E_cd] = 0
```

This follows directly from the half-derivation axiom:
`2·D([x,y]) = [D(x), y] + [x, D(y)]`

When `[x,y] = 0`: LHS = `2·D(0) = 0`, so RHS = 0. ✓

## Architecture

A new file `E/HalfDerivation/Semantics.lean` sits between Bracket and ToNCoeff:

```
Infrastructure → Bracket → Semantics (NEW) → ToNCoeff → Adjacent → Classification
```

It defines:

| Definition | Meaning |
|-----------|---------|
| `bracketLeftPos` (A) | `coeffOf D i j u c` when `u<c ∧ v=d` |
| `bracketLeftNeg` (B) | `coeffOf D i j d v` when `u=c ∧ d<v` |
| `bracketRightPos` (C) | `coeffOf D c d j v` when `u=i ∧ j<v` |
| `bracketRightNeg` (D) | `coeffOf D c d u i` when `u<i ∧ v=j` |
| `bracketLeft` | A − B = coeff of E_uv in [D(E_ij), E_cd] |
| `bracketRight` | C − D = coeff of E_uv in [E_ij, D(E_cd)] |
| `bracketIdentity` | A − B + C − D |

## Key Lemma (work in progress)

```lean4
lemma half_deriv_bracket_zero_nonadj (D : HalfDerivation F n)
    (h_nonadj : j - i ≥ 2) (hj_ne_c : j ≠ c) (hd_ne_i : d ≠ i) :
    A − B + C − D = 0
```

This replaces `coeffOf_nonadjacent` in the non-adjacent branches of
`coeffOf_cond_zero`. The proof uses `half_deriv_cond` directly — no
adjacent classification needed.

## Refactoring `coeffOf_cond_zero`

After the Semantics bridge is complete, `coeffOf_cond_zero` becomes:

```lean4
lemma coeffOf_cond_zero (...) : A − B + C − D = 0 := by
  by_cases h_nonadj_ij : j - i ≥ 2
  · exact half_deriv_bracket_zero_nonadj D ... h_nonadj_ij ... hj_ne_c hd_ne_i
  · -- adjacent source: handle separately (single point, not scattered across 16 branches)
    ...
```

## Principle

> The bracket identity is an **operator-level theorem** that should be proved
> from the half-derivation axiom, not a coefficient-level theorem proved by
> individual coefficient vanishing. The 16 `split_ifs` cases are just the
> expansion of the operator identity into basis coefficients — they should
> not carry the proof burden.

This aligns with the project's overall style: Recursion.lean proves
structural identities, Adjacent.lean proves support restrictions,
ImageInCenter.lean proves image containment. Bracket.lean should be a
**local combinatorial identity library**, not a miniature Classification.lean.
