# Structural Classification Methodology for Lean Research

When formalizing or discovering mathematical theorems in Lean, this methodology
prioritizes structural (quotient-space) arguments over coordinate-level case analysis.

## Core Principle

Prove structural properties first: `T([L,L]) = 0` and `Im(T) ⊆ Z(L)`, then deduce
the classification. This is cleaner than proving each coefficient individually.

## When to Apply

When the target is a classification theorem of the form:

    Der(L) ≅ F·Id ⊕ Hom(L/[L,L], Z(L))

or more generally, when the space of derivations/operators splits into a scalar
part and a "vanishes on derived" + "lands in center" part.

## Step-by-Step Methodology

### Phase 1: Build the Classification Skeleton First

Create a `Classification.lean` file with theorem shells BEFORE filling in proofs:

```lean4
def scalarPart (D : HalfDerivation F n) : F := ...
def T (D : HalfDerivation F n) : HalfDerivation F n := D  -- placeholder

theorem T_derived_zero (D : HalfDerivation F n) (hn5 : 5 ≤ n) :
    ∀ (x : MatIdx n), x.j - x.i ≥ 2 → (T F n D).coeff x x = 0 := by
  sorry  -- From: Diagonalization + classification_n_ge_5

theorem T_image_in_center (D : HalfDerivation F n) (hn5 : 5 ≤ n) :
    (T F n D).coeff ∈ centerSubspace F n := by
  sorry  -- Depends on Adjacent.lean

theorem T_factors_through_abelianization (D : HalfDerivation F n) (hn5 : 5 ≤ n) : True := by
  trivial

theorem classification (hn5 : 5 ≤ n) : True := by
  trivial
```

**Benefits:** Each lemma knows its purpose. You can see what Adjacent.lean needs to
provide before writing it. The skeleton compiles and passes `lake build` immediately.

### Phase 2: Prove T_derived_zero

This is usually the simpler of the two main theorems. For the N_n case:

- Nonadjacent sources (j-i ≥ 2) are already classified by `Diagonalization.nonadjacent`
- All diagonal coefficients are equal by `RecursiveSystem.classification_n_ge_5`
- Therefore `D(E(i,j)) = c·E(i,j)` for all nonadjacent (i,j)
- Hence `T(E(i,j)) = (D - c·Id)(E(i,j)) = 0`
- Since `[L,L]` is spanned by such `E(i,j)`, `T([L,L]) = 0`

### Phase 3: Prove T_image_in_center

This is the hard part and typically depends on an Adjacent.lean lemma:

```lean4
lemma adjacent_reduce_to_corner (D : HalfDerivation F n) (i : ℕ)
    (hi : 1 ≤ i) (hin : i+1 ≤ n) (y : MatIdx n)
    (h_neq : y.i ≠ 1 ∨ y.j ≠ n) : (T D).coeff (E(i,i+1)) y = 0 := ...
```

Proving this lemma requires analyzing the off-diagonal coefficients of
adjacent sources:
- Type A (u,i+1): eliminated via right bracket + descending induction
- Type B (i,v): eliminated via left bracket + ascending induction
- Interior Type C (u,v): eliminated via cond_zero with carefully chosen (c,d)
- Boundary Type C (v=n, u>1): cond_zero with (c,d) = (1,u)

### Phase 4: Deduce the Quotient Map

From T_derived_zero, T descends to a map L/[L,L] → L.
From T_image_in_center, the image lands in Z(L).
Compose to get T_induced : L/[L,L] → Z(L).

### Phase 5: Dimension Count

dim Der(L) = dim(F·Id) + dim(Hom(L/[L,L], Z(L)))
           = 1 + dim(L/[L,L]) · dim(Z(L))

For N_n: dim(L/[L,L]) = n-1, dim(Z(L)) = 1, so dim = n.

## Signs That a Structural Approach Is Better

- You're writing a lemma that only makes sense for a specific coordinate (i,j)
- You have multiple type cases (Type A, B, C) with separate lemmas
- Cross-terms between cases create long mutual-dependency chains
- The paper you're formalizing has an "obviously, all other terms vanish" step
- **You find yourself calling `coeffOf_nonadjacent` (or any coefficient-vanishing lemma)
  from within a bracket-expansion identity.** This is the anti-pattern that created the
  `coeffOf_cond_zero` circular dependency. The identity `[D(E_ij), E_cd] + [E_ij, D(E_cd)] = 0`
  already forces the sum to zero — individual coefficient killing is redundant and creates cycles.
  See `references/coeffOf-cond-zero-strategy-C.md` for the full resolution.

## Pitfall: individual coefficient killing inside bracket identities

When proving a bracket-expansion identity like `coeffOf_cond_zero` (which encodes
`[D(E_ij), E_cd] + [E_ij, D(E_cd)] = 0` when `[E_ij, E_cd] = 0`), the temptation is:

```
split_ifs → surviving term: coeffOf D i j u c → call coeffOf_nonadjacent → = 0
```

This is wrong for two reasons:
1. It creates a cycle: `coeffOf_nonadjacent` inherits an adjacent-source gap that
   downstream classification modules (which depend on `coeffOf_cond_zero`) would resolve
2. It treats each surviving term individually rather than using the bracket identity
   itself to force the sum to zero

The correct approach: prove the 4-term sum identity directly through bracket decomposition,
using support analysis (which terms can produce E_uv) rather than coefficient analysis
(which terms happen to be nonzero).

## When Coordinate-Level Is Still Needed

Structural proofs need coordinate-level lemmas as building blocks:
- The right/left bracket expansions for Type A/B elimination
- The cond_zero term selection for Type C elimination
- Boundary case analysis for the corner coefficient

The methodology is: **prove the structure theorem shell first, then fill in the
coordinate-level lemmas as needed by the shells.**
