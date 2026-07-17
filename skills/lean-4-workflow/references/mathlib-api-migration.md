# Mathlib API Migration Notes (v4.31.0-rc1)

Common API changes encountered when porting code written against an earlier
mathlib version.  These were discovered during a 2026-06-29 release audit of
the `e/` Lean project.

## Renamed / Removed Lemmas

| Old name | New name | Notes |
|----------|----------|-------|
| `finiteDimensional_of_finrank` | `FiniteDimensional.of_finrank_pos` | Takes `0 < finrank` instead of `finrank ≠ 0` |
| `Submodule.disjoint_iff` | `Submodule.disjoint_def` | Same type |
| `Submodule.finrank_sup_eq_of_disjoint` | `finrank_sup_add_finrank_inf_eq` + `eq_bot` | The old lemma doesn't exist; use `finrank_sup_add_finrank_inf_eq s t` then `rw [h_disjoint.eq_bot, finrank_bot, add_zero]` |
| `finrank_span_eq_card` | Still exists | But requires `LinearIndependent` proof; for singletons prefer `finrank_span_singleton` |

## Import Paths

| Module | Correct import |
|--------|---------------|
| `FiniteDimensional` lemmas | `import Mathlib.LinearAlgebra.FiniteDimensional.Lemmas` |
| `FiniteDimensional` basics | `import Mathlib.LinearAlgebra.FiniteDimensional.Basic` |

Note: `import Mathlib.LinearAlgebra.FiniteDimensional` does NOT work —
`FiniteDimensional` is a directory, not a single module.

## Working Patterns

### `finrank` of a span of a single nonzero vector

```lean
have h_finrank : Module.finrank F (Submodule.span F {v}) = 1 := by
  have h_nonzero : v ≠ 0 := ...
  rw [finrank_span_singleton h_nonzero]
```

Much simpler than using `finrank_span_eq_card` with a `LinearIndependent` proof.

### `coeffOf` expansion without validity proof boilerplate

When you need `coeffOf D 1 2 1 2` and have `hn3 : 3 ≤ n` (so `2 ≤ n`), use:

```lean
have hpos : coeffOf (halfDeriv_Id (F := F) (n := n)) 1 2 1 2 = (1 : F) := by
  have h2n : 2 ≤ n := by omega
  unfold coeffOf
  simp [halfDeriv_Id, coeff_Id_valid, h2n]
```

This is more reliable than `coeffOf_f` with 6 omega arguments, because:
- `coeffOf_f` may not reduce due to implicit/explicit binder mode confusion
- The `unfold coeffOf; simp` approach works consistently

### `coeffOf` direction after `split_ifs`

When proving `coeffOf D = f`, the goal after `unfold coeffOf; split_ifs` is
`0 = f i j u v` for invalid-index branches.  But lemmas like `IsValidSupp`
return `f i j u v = 0`.  **Always use `.symm`:**

```lean
  · rcases h with (hi | hij | hjn | hu | huv | hvn)
    · exact (k.property.2.1 i j u v h_invalid).symm
    ...
```

## Known Pitfalls

### `Max Type` / `Min Type` with `Submodule.finrank_sup_add_finrank_inf_eq`

The lemma `Submodule.finrank_sup_add_finrank_inf_eq` may fail with
`failed to synthesize instance of type class Max Type` when the ambient
vector space is `ℕ → ℕ → ℕ → ℕ → F` (a large function space).  This appears
to be a typeclass resolution issue in mathlib v4.31.0-rc1.  Workarounds:

1. Use `rw [← Submodule.finrank_sup_add_finrank_inf_eq s t, h_disjoint.eq_bot,
   finrank_bot, add_zero]` to rewrite the goal directly (instead of getting a hypothesis)
2. Provide explicit `[FiniteDimensional F s] [FiniteDimensional F t]` instances
   via `FiniteDimensional.of_finrank_pos`
3. If both fail, consider computing the sup finrank via lower+upper bound instead

### `FiniteDimensional` instances only needed for `finrank_sup` lemmas

The `FiniteDimensional` typeclass is NOT required for `Module.finrank` to
return a nonzero value (it returns 0 for infinite-dimensional).  But
`finrank_sup_add_finrank_inf_eq` requires both subspaces to be
`FiniteDimensional`.  The standard way to obtain this:

```lean
haveI : FiniteDimensional F (kerPhi ...) :=
  FiniteDimensional.of_finrank_pos (by rw [h_finrank_ker]; omega)
```

## Structure Instance Pitfalls

### Adding `AddCommMonoid` and `Module` to a custom structure

When a structure has standalone `Add`, `Zero`, `Neg`, `Sub`, `SMul` instances
but lacks bundled typeclasses, `LinearMap` operations (like `→ₗ[F]`) will fail
with "failed to synthesize instance of type class AddCommMonoid".

**Fix**: Add `AddCommMonoid` and `Module` instances. Since the structure's fields
are pointwise operations on `F`, use a `coeff_inj` helper:

```lean
private lemma coeff_inj {F n} [Field F] [CharNeTwo F] {D1 D2 : HalfDerivation F n}
    (h : D1.coeff = D2.coeff) : D1 = D2 := by
  rcases D1 with ⟨c1, hl1⟩; rcases D2 with ⟨c2, hl2⟩; subst h; rfl
```

**Also required**: a `@[simp]` lemma for `coeff_zero`:
```lean
@[simp] lemma coeff_zero (i j u v : ℕ) : (0 : HalfDerivation F n).coeff i j u v = 0 := rfl
```

### `coeffOf` direction pitfall

In `kerPhi`/`IsValidSupp` proofs, the goal after `unfold coeffOf; split_ifs` is
`0 = (k : V F) i j u v`, but `k.property.2.1` returns `(k : V F) i j u v = 0`.
Always add `.symm` to the 6 invalid-index branches.
