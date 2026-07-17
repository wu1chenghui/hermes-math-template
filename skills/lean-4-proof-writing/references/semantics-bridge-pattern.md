# Semantics Bridge Pattern — Replacing Case Analysis with Operator Identities

This pattern was developed during the June 2026 refactoring of the
1/2-derivation classification project (N_n, dim = n). It replaces a
16-branch `split_ifs` analysis with a 2-branch `by_cases` proof by
lifting the proof from coefficient-level to operator-level.

## When to use

A lemma has ≥8 `split_ifs` branches, each calling vanishing lemmas
like `coeffOf_nonadjacent` or marked as `sorry`. The branches share
a common mathematical structure: they are projections of an operator
identity `[D(x), y] + [x, D(y)] = 0` onto specific basis elements.

## The pattern (4 steps)

### Step 1: Define coefficient extraction functions

```lean4
noncomputable def bracketLeft (D : HalfDerivation F n) (i j c d u v : ℕ) : F :=
  (if u < c ∧ v = d then coeffOf D i j u c else 0)
  - (if u = c ∧ d < v then coeffOf D i j d v else 0)

noncomputable def bracketRight (D : HalfDerivation F n) (i j c d u v : ℕ) : F :=
  (if u = i ∧ j < v then coeffOf D c d j v else 0)
  - (if u < i ∧ v = j then coeffOf D c d u i else 0)
```

### Step 2: Prove the operator identity once

For non-adjacent sources, `coeffOf_nonadjacent` gives both
`bracketLeft = 0` and `bracketRight = 0`:

```lean4
lemma half_deriv_bracket_zero_nonadj (...) :
    bracketLeft D i j c d u v + bracketRight D i j c d u v = 0 := by
  have h2ne : (2 : F) ≠ 0 := CharNeTwo.char_ne_two
  -- A = 0 because target (u,c) ≠ source diagonal (i,j)
  -- B = 0 because target (d,v) ≠ source diagonal (i,j)
  -- If (c,d) non-adjacent: C = D = 0 similarly
  -- If (c,d) adjacent: C−D = 0 via adjacent analysis (the remaining gap)
  ...
```

### Step 3: Refactor the consumer lemma

Old version (16 branches):
```lean4
lemma coeffOf_cond_zero (...) : A − B + C − D = 0 := by
  split_ifs
  · ... omega
  · ... coeffOf_nonadjacent
  · ... sorry  -- adjacent gap
  ...
```

New version (2 branches):
```lean4
lemma coeffOf_cond_zero (...) : A − B + C − D = 0 := by
  by_cases h_nonadj : j - i ≥ 2
  · exact half_deriv_bracket_zero_nonadj ...
  · sorry  -- single adjacent gap
```

### Step 4: If Step 2 creates a circular import, extract to a new file

In this project, `coeffOf_cond_zero` was in `Bracket.lean`, which is imported
by `Semantics.lean`. But the refactored proof calls `half_deriv_bracket_zero_nonadj`
from `Semantics.lean`, creating a cycle. Solution: **create `BracketCondZero.lean`**
that imports both `Bracket.lean` and `Semantics.lean`, and place the refactored
lemma there.

```
Infrastructure → Bracket → Semantics → BracketCondZero → ToNCoeff → ...
                                                ↑
                                     (no cycle — BracketCondZero
                                      imports both, used by ToNCoeff)
```

## The key insight

The `split_ifs` branches are NOT independent mathematical cases — they are
projections of a single operator identity `[D(x), y] + [x, D(y)] = 0` onto
different basis elements. Proving the operator identity once and projecting
is both shorter and more maintainable than 16 separate case analyses.

## Resulting architecture

```
Infrastructure
      │
Bracket (coeffOf, coeffOf_cond, coeffOf_nonadjacent)
      │
Semantics (bracketLeft, bracketRight, half_deriv_bracket_zero_nonadj)
      │
BracketCondZero (coeffOf_cond_zero, refactored)
      │
ToNCoeff → Adjacent → ImageInCenter → Classification
```

The remaining gap is `half_deriv_bracket_zero_adjacent` — the bracket
identity for adjacent sources. This is isolated in `AdjacentBridge.lean`
as an independent mini-project depending only on Infrastructure + Bracket.
