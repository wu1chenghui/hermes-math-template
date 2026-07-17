# Adding `AddCommMonoid` + `Module` Instances to a Custom Structure

## When

A structure (like `HalfDerivation`) has standalone `Add`, `Zero`, `Neg`, `Sub`,
`SMul` instances but no bundled typeclasses (`AddSemigroup`, `AddMonoid`,
`AddCommMonoid`, `Module`).  Code that needs `LinearMap` (via `→ₗ[F]`) or
algebraic operations (`abel`, `simp` with `add_comm`) will fail with
`failed to synthesize instance of type class AddCommMonoid ...`.

## The `coeff_inj` pattern

For a structure where equality is determined by a single data field (e.g.
`coeff : ℕ⁴ → F`), prove a private injectivity lemma first:

```lean
private lemma coeff_inj {F n} [Field F] [CharNeTwo F]
    {D1 D2 : HalfDerivation F n} (h : D1.coeff = D2.coeff) : D1 = D2 := by
  rcases D1 with ⟨c1, hl1⟩
  rcases D2 with ⟨c2, hl2⟩
  subst h
  rfl
```

Key: use **implicit** `{F n}` and explicit `[Field F] [CharNeTwo F]` typeclass
arguments.  This ensures the lemma works in any context where the typeclass
instances are available, without requiring explicit `(n := n)` at call sites.

## The instances

```lean
instance : AddCommMonoid (HalfDerivation F n) where
  add_assoc D1 D2 D3 := coeff_inj (by
    ext i j u v
    simp [coeff_add, add_assoc])
  zero_add D := coeff_inj (by
    ext i j u v
    simp [coeff_add, coeff_zero])
  add_zero D := coeff_inj (by
    ext i j u v
    simp [coeff_add, coeff_zero])
  add_comm D1 D2 := coeff_inj (by
    ext i j u v
    simp [coeff_add, add_comm])
  nsmul := nsmulRec

instance : Module F (HalfDerivation F n) where
  add_smul r s D := coeff_inj (by
    ext i j u v
    simp [coeff_add, coeff_smul, add_mul])
  zero_smul D := coeff_inj (by
    ext i j u v
    simp [coeff_smul, coeff_zero])
  smul_zero r := coeff_inj (by
    ext i j u v
    simp [coeff_smul, coeff_zero])
  smul_add r D1 D2 := coeff_inj (by
    ext i j u v
    simp [coeff_add, coeff_smul, mul_add])
  mul_smul r s D := coeff_inj (by
    ext i j u v
    simp [coeff_smul, mul_assoc])
  one_smul D := coeff_inj (by
    ext i j u v
    simp [coeff_smul])
```

## Prerequisites

You need `@[simp]` lemmas for the pointwise operations:

```lean
@[simp] lemma coeff_add (D1 D2 : HalfDerivation F n) (i j u v : ℕ) :
    (D1 + D2).coeff i j u v = D1.coeff i j u v + D2.coeff i j u v := rfl

@[simp] lemma coeff_smul (c : F) (D : HalfDerivation F n) (i j u v : ℕ) :
    (c • D).coeff i j u v = c * D.coeff i j u v := rfl

@[simp] lemma coeff_zero (i j u v : ℕ) : (0 : HalfDerivation F n).coeff i j u v = 0 := rfl
```

Without `coeff_zero`, `simp` won't reduce `(0 : HalfDerivation ...).coeff ...` to `0`
in implicit-binder contexts (where the `variable (F n)` vs `variable {F n}` boundary
breaks definitional reduction).

## Pitfalls

- `rcases D with ⟨c, hl⟩; simp` alone does NOT work — the goal is a structural
  equality (`D1 = D2`), and `simp` doesn't know the structure's `Prop` field is
  irrelevant.  Use `coeff_inj` to reduce to pointwise equality.
- `coeff_inj` MUST have `{F n}` as implicit binders.  If `F` or `n` are explicit
  (from a `variable (F n)` section), Lean can't match the goal type, giving
  `Type mismatch: coeff_inj has type ∀ (n : ℕ) ... but expected 1 • D = D`.
- The `Module` instance requires ALL 6 fields: `add_smul`, `zero_smul`,
  `smul_zero`, `smul_add`, `mul_smul`, `one_smul`.  Missing any gives
  `Fields missing` error.

## Import

Adding these instances to the structure's defining file (e.g. `E/Core/HalfDerivation.lean`)
avoids import issues.  The imported `Mathlib.Tactic` provides `nsmulRec`.
`add_mul` and `mul_assoc` come from `Field` via standard imports.
