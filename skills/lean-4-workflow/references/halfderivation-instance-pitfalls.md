# HalfDerivation Instance Pitfalls

## Missing `AddCommMonoid` and `Module` Instances

`HalfDerivation` in `E/Core/HalfDerivation.lean` only has standalone instances:
- `Zero`
- `Add`
- `Neg`
- `Sub`
- `SMul F`

It does NOT have bundled algebraic typeclasses:
- `AddSemigroup`
- `AddMonoid`
- `AddCommMonoid`
- `Module F`

This means `LinearMap` (via `→ₗ[F]`) fails with:
```
failed to synthesize instance of type class AddCommMonoid (HalfDerivation F n)
```

## Fix Pattern

Add instances using a `coeff_inj` helper (since `HalfDerivation` has no `@[ext]` lemma):

```lean
private lemma coeff_inj {F n} [Field F] [CharNeTwo F]
    {D1 D2 : HalfDerivation F n} (h : D1.coeff = D2.coeff) : D1 = D2 := by
  rcases D1 with ⟨c1, hl1⟩
  rcases D2 with ⟨c2, hl2⟩
  subst h
  rfl

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

Also add `coeff_zero` lemma (needed because `Zero` instance was defined under
`variable (F n)` explicit binders, making definitional reduction unreliable in
`variable {F n}` contexts):

```lean
@[simp] lemma coeff_zero (i j u v : ℕ) : (0 : HalfDerivation F n).coeff i j u v = 0 := rfl
```

## Key Points

- `coeff_inj` must have `{F n}` implicit (NOT from section `variable (F n)`)
  and `[Field F] [CharNeTwo F]` typeclass arguments — otherwise it can't match
  goals in `Module`/`AddCommMonoid` instance contexts.
- `ext i j u v` works because `D.coeff` is a Pi type (`ℕ → ℕ → ℕ → ℕ → F`),
  which has `funext`-based `@[ext]`.
- Use `coeff_add`, `coeff_smul`, `coeff_zero` as simp lemmas.
- For `add_smul`: use `add_mul` (not `add_smul` which is about module scalar addition).
- The proofs are pointwise because all operations on `HalfDerivation` are pointwise
  on the `coeff` field and `F` is a field.
