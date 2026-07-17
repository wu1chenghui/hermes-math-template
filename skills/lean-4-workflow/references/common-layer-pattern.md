# Common Layer Pattern — Shared Theorems Across Multiple Structures

Discovered during the 1/2-derivation classification project (2026-06-19).

## Problem

Two structures (`NnDerivation`, `HalfDerivation`) share a subset of theorems
(ONLY, Pair) that only depend on a common property (`bracket_zero` for commuting
sources), but differ in other theorems (Chain, Bridge) that depend on
structure-specific axioms (`coeffOf_cond` with/without factor 2).

Without a common layer, the shared theorems get duplicated or tied to one
structure.

## Solution: ProjectionIdentity pattern

### Step 1: Extract the common interface as a structure

```lean4
structure ProjectionIdentity (F : Type) [Field F] where
  n : ℕ
  coeff : ℕ → ℕ → ℕ → ℕ → F
  bracket_zero (i j c d u v : ℕ)
    (hi : 1 ≤ i) (hij : i < j) (hjn : j ≤ n)
    (hc : 1 ≤ c) (hcd : c < d) (hdn : d ≤ n)
    (hu : 1 ≤ u) (huv : u < v) (hvn : v ≤ n)
    (hj_ne_c : j ≠ c) (hi_ne_d : i ≠ d) :
    bracketIdentity coeff i j c d u v = 0
```

### Step 2: Define shared helpers on the structure

```lean4
namespace ProjectionIdentity

def coeffOf (pi : ProjectionIdentity F) (i j u v : ℕ) : F :=
  if ... pi.n ... then pi.coeff i j u v else 0

lemma coeffOf_f (pi : ProjectionIdentity F) ... : coeffOf pi i j u v = pi.coeff i j u v := ...

theorem bracket_zero_coeff (pi : ProjectionIdentity F) ... : bracketIdentity pi.coeff ... = 0 :=
  pi.bracket_zero ...
```

### Step 3: Add projection functions to each source structure

```lean4
-- In Derivation.lean:
namespace NnDerivation
def toProjectionIdentity (D : NnDerivation F n) : ProjectionIdentity F where
  n := n
  coeff := D.coeff
  bracket_zero := fun ... => bracket_zero D ...
```

### Step 4: Parameterize shared theorems on ProjectionIdentity

```lean4
-- Only.lean: no longer imports NnDerivation
theorem only_A (pi : ProjectionIdentity F) (i c d u v : ℕ) ... :
    coeffOf pi i (i+1) u c = 0 := ...
```

### Step 5: Use via `.toProjectionIdentity`

```lean4
-- For HalfDerivation D:
let pi := D.toProjectionIdentity
have h := only_A pi ...  -- works!

-- For NnDerivation D:
let pi := D.toProjectionIdentity
have h := only_A pi ...  -- same theorem!
```

## When to use

This pattern applies when:

1. Two or more structures share a subset of theorems based on a common property
2. Those theorems don't depend on structure-specific axioms
3. The common property can be expressed as a function `F → F → ...` (no Prop-only fields)

**Use `structure` (not `class`)** when the common interface carries data
(like `n : ℕ`) that's needed at runtime in if-guards or computations.
Typeclasses can only resolve parameters inferable from the type signature.

## Result in our project

| Layer | What | Theorems | Applies to |
|-------|------|----------|------------|
| Common | `ProjectionIdentity` | 8 (ONLY x4, Pair x4) | NnDerivation + HalfDerivation |
| Specific | `NnDerivation` | 19 (Chain, Bridge, Diagonal) | Full derivations only |
| Specific | `HalfDerivation` | 3+ (Chain/Bridge TBD) | 1/2-derivations only |

## Key insight

The commuting-source bracket identity (`bracketIdentity = 0` when j≠c, i≠d)
is identical for both full and half derivations. This was verified mathematically:

- Full: `D([E,E])=0 → [D(E),E]+[E,D(E)]=0 → bracketIdentity=0`
- Half: `2δ([E,E])=0 → [δ(E),E]+[E,δ(E)]=0 → bracketIdentity=0`

Therefore ONLY and Pair (which only use this condition) are universal.
Chain and Bridge use `coeffOf_cond` which differs by factor 2 between
architectures, so they stay architecture-specific.
