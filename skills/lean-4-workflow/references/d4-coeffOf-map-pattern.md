# D4 finrank_halfDer — coeffOf_map Pattern

## The problem

The `HalfDerivation` type's raw `coeff : ℕ⁴ → F` carries junk at invalid indices
(any `(i,j,u,v)` outside the 6-constraint valid range). This makes `finrank` on
the raw `HalfDerivation` space ill-defined — there are infinitely many invalid
index tuples, each a degree of freedom if we count the raw coeff table.

The original plan was to build an `Equiv` (or `LinearEquiv`) between
`HalfDerivation` and `F × kerΦ`, but this required a `Module` instance on
`HalfDerivation` (which didn't exist) and `LinearEquiv` blew up on missing
`AddCommMonoid`.

## The solution

Define a *canonical linear map* that sends each `HalfDerivation` to its
`coeffOf` image (which is zero at invalid indices by construction):

```lean
def coeffOf_map : HalfDerivation F n →ₗ[F] V F where
  toFun D i j u v := coeffOf D i j u v
  map_add' D1 D2 := by
    ext i j u v : 4
    simp [HalfDerivation.coeffOf_add]
  map_smul' c D := by
    ext i j u v : 4
    simp [HalfDerivation.coeffOf_smul]
```

Then prove:

```lean
theorem coeffOf_range_eq_span_id_sup_kerPhi :
    LinearMap.range coeffOf_map = span{coeffOf(Id)} ⊔ kerΦ
```

The centered-part decomposition gives one direction; the reverse uses
`kerPhi_lift` to embed `kerΦ` elements back into `HalfDerivation`.

Then the final theorem:

```lean
theorem finrank_halfDer :
    Module.finrank F (LinearMap.range coeffOf_map) = n + 5 := by
  rw [coeffOf_range_eq_span_id_sup_kerPhi]
  -- 1-dim orthogonal complement: span{coeffOf(Id)} ∩ kerΦ = ⊥
  have h_disjoint : Disjoint sId kerΦ := ...
  -- Use finrank_sup_add_finrank_inf_eq (mathlib)
  -- finrank(sId ⊔ kerΦ) = finrank(sId) + finrank(kerΦ)  [disjoint → inf = ⊥]
  --                      = 1 + (n+4) = n+5
```

## Why this works

- `coeffOf` is defined per `HalfDerivation` in `HalfProjection.lean` using
  nested `if` statements that zero out invalid indices
- `coeffOf_map` is a true `LinearMap` (requires `AddCommMonoid` + `Module`
  instances on `HalfDerivation` — added in HalfDerivation.lean)
- The `range` of `coeffOf_map` is the *canonical image* — a finite-dimensional
  subspace of `V F = ℕ⁴ → F` where all invalid-index values are exactly zero
- This avoids the Equiv decomposition entirely

## Dependencies added to HalfDerivation.lean

Two new instances needed (added 2026-06-29):

```lean
instance : AddCommMonoid (HalfDerivation F n) where ...
instance : Module F (HalfDerivation F n) where ...
```

Plus a helper lemma `coeff_zero` (analogous to existing `coeff_add`,
`coeff_smul`, `coeff_neg`):

```lean
@[simp] lemma coeff_zero (i j u v : ℕ) : (0 : HalfDerivation F n).coeff i j u v = 0 := rfl
```

And a `coeff_inj` lemma to reduce HalfDerivation equality to coeff equality
(used in the instance proofs):

```lean
private lemma coeff_inj {D1 D2 : HalfDerivation F n} (h : D1.coeff = D2.coeff) : D1 = D2 := ...
```

## Key gotchas during implementation

1. **`ext` works on Pi types**: `D1.coeff = D2.coeff` is `ℕ⁴ → F` equality,
   so `ext i j u v` works (Pi has built-in `@[ext]` via `funext`).

2. **`coeff_inj` binder mode**: must use `{F n}` (implicit) when the lemma
   is called from instance proofs because the expected goal doesn't provide
   `F`/`n` explicitly.

3. **mathlib API changes** (v4.31.0-rc1):
   - `Submodule.disjoint_iff` → `Submodule.disjoint_def`
   - `Submodule.finrank_sup_eq_of_disjoint` → removed; use
     `finrank_sup_add_finrank_inf_eq` + `Disjoint.eq_bot` + `finrank_bot`

4. **`finrank_span_eq_card`** takes `LinearIndependent` proof, not `h ≠ 0`.
   For a singleton nonzero vector, use `linearIndependent_unique` or direct
   construction.
