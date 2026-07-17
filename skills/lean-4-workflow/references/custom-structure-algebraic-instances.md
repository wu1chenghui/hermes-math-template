# Adding Algebraic Instances to Custom Lean Structures

## Problem

A custom structure (e.g., `HalfDerivation`) has standalone `Add`, `Zero`, `SMul`
instances but NOT bundled typeclasses (`AddCommMonoid`, `Module`). This blocks
`LinearMap` operations (`D →ₗ[F] V`) and `abel`/`ring` tactics.

## Solution: Prove instances pointwise via `coeff_inj`

Step 1: Add a `coeff_zero` `@[simp]` lemma:

```lean
@[simp] lemma coeff_zero (i j u v : ℕ) : (0 : HalfDerivation F n).coeff i j u v = 0 := rfl
```

Step 2: Add a `coeff_inj` helper (private, implicit `{F n}` with typeclass constraints):

```lean
private lemma coeff_inj {F n} [Field F] [CharNeTwo F]
    {D1 D2 : HalfDerivation F n} (h : D1.coeff = D2.coeff) : D1 = D2 := by
  rcases D1 with ⟨c1, hl1⟩
  rcases D2 with ⟨c2, hl2⟩
  subst h
  rfl
```

**Critical**: use `{F n}` (implicit) NOT `(F n)` (explicit). The implicit version
lets `coeff_inj` match the goal shape in `instance` proofs where `F` and `n` are
section variables.

Step 3: Define `AddCommMonoid` and `Module` instances:

```lean
instance : AddCommMonoid (HalfDerivation F n) where
  add_assoc D1 D2 D3 := coeff_inj (by
    ext i j u v
    simp [coeff_add, add_assoc])
  zero_add D := coeff_inj (by ext i j u v; simp [coeff_add, coeff_zero])
  add_zero D := coeff_inj (by ext i j u v; simp [coeff_add, coeff_zero])
  add_comm D1 D2 := coeff_inj (by
    ext i j u v
    simp [coeff_add, add_comm])
  nsmul := nsmulRec

instance : Module F (HalfDerivation F n) where
  add_smul r s D := coeff_inj (by
    ext i j u v
    simp [coeff_add, coeff_smul, add_mul])
  zero_smul D := coeff_inj (by ext i j u v; simp [coeff_smul, coeff_zero])
  smul_zero r := coeff_inj (by ext i j u v; simp [coeff_smul, coeff_zero])
  smul_add r D1 D2 := coeff_inj (by
    ext i j u v
    simp [coeff_add, coeff_smul, mul_add])
  mul_smul r s D := coeff_inj (by
    ext i j u v
    simp [coeff_smul, mul_assoc])
  one_smul D := coeff_inj (by ext i j u v; simp [coeff_smul])
```

The `ext i j u v` works because `D.coeff : ℕ → ℕ → ℕ → ℕ → F` is a Pi type,
so `ext` applies `funext` automatically. The structure's `Prop` field
(`half_leibniz`) is ignored because `coeff_inj` only requires `coeff` equality.

## Pitfalls

- `simp` alone won't close structural equality goals. Must use `coeff_inj` first.
- `coeff_inj` MUST use `{F n}` with `[Field F] [CharNeTwo F]` — explicit binders
  cause type mismatch in `instance` proofs (expected `1 • D = D` but got
  `∀ n, coeff ... = coeff ... → ...`).
- The `coeff_zero` lemma needs to be in a `variable (F n)` section (explicit) for
  `rfl` to work, because definitional reduction of structure projections fails
  under `variable {F n}` (implicit).
