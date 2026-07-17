# Mathlib `finrank_sup` API Compatibility (v4.31.0-rc1)

## Problem: `Max Type` error from `Submodule.finrank_sup_add_finrank_inf_eq`

When computing `finrank(s ⊔ t)` for subspaces of `V = ℕ → ℕ → ℕ → ℕ → F` (a Pi type),
the lemma `Submodule.finrank_sup_add_finrank_inf_eq` can trigger:

```
failed to synthesize instance of type class Max Type
```

## Root Cause

The `rank` (Cardinal) version triggers `Max`/`Min` universe-level typeclass resolution
for Pi types like `ℕ → ℕ → ℕ → ℕ → F`. This is a mathlib4 universe inference issue
that does NOT occur in `lean_run_code` but DOES occur in multi-import project contexts.

## Solution

Use the **finrank** (ℕ) version, NOT the rank (Cardinal) version:

```lean
import Mathlib.LinearAlgebra.FiniteDimensional.Basic
import Mathlib.LinearAlgebra.FiniteDimensional.Lemmas

-- ✅ Works (ℕ version):
have h_eq := Submodule.finrank_sup_add_finrank_inf_eq s t

-- ❌ Fails (Cardinal version):
have h_rank := Submodule.rank_sup_add_rank_inf_eq s t
```

Both `Basic` AND `Lemmas` imports are needed:
- `Basic` provides `FiniteDimensional.of_finrank_pos`
- `Lemmas` provides `Submodule.finrank_sup_add_finrank_inf_eq`

The `FiniteDimensional` instances are obtained via:
```lean
haveI : FiniteDimensional F s :=
  FiniteDimensional.of_finrank_pos (by rw [h_finrank_s]; omega)
haveI : FiniteDimensional F t :=
  FiniteDimensional.of_finrank_pos (by rw [h_finrank_t]; omega)
```

## Critical Pitfall: `LinearEquiv.finiteDimensional` direction (2026-06-30)

`LinearEquiv.finiteDimensional` goes **domain → codomain**:

```lean
-- f : V ≃ₗ[K] V₂
-- f.finiteDimensional      : FiniteDimensional K V  → FiniteDimensional K V₂  (✓ domain → codomain)
-- f.symm.finiteDimensional : FiniteDimensional K V₂ → FiniteDimensional K V   (✗ opposite direction)
```

When constructing `FiniteDimensional` instances for Submodules via LinearEquiv:

```lean
-- To prove FiniteDimensional K sId, build h_equiv : F ≃ₗ[K] sId
have h_equiv : F ≃ₗ[K] sId := ...
-- ❌ WRONG: h_equiv.symm.finiteDimensional requires FD(sId) → we're trying to PROVE that
-- ✅ CORRECT: h_equiv.finiteDimensional requires FD(F) which is trivially available
haveI : FiniteDimensional K sId := h_equiv.finiteDimensional inferInstance
```

Common pattern: when the target Submodule is equated to `F` (or another
known-finite-dimensional space), the `LinearEquiv` goes FROM the known space
TO the Submodule.  Always use the **forward** direction (NOT `.symm`).

**Symptom when wrong**: "failed to synthesize instance of type class Max Type"
at the subsequent `Submodule.finrank_sup_add_finrank_inf_eq` call — this is a
cascade from a missing `FiniteDimensional` instance, NOT a universe issue.

```lean
have h_fin : Module.finrank F (s ⊔ t) = n + m := by
  haveI : FiniteDimensional F s :=
    FiniteDimensional.of_finrank_pos (by rw [h_fin_s]; omega)
  haveI : FiniteDimensional F t :=
    FiniteDimensional.of_finrank_pos (by rw [h_fin_t]; omega)
  have h_eq := Submodule.finrank_sup_add_finrank_inf_eq s t
  rw [h_disjoint.eq_bot, finrank_bot, add_zero] at h_eq
  rw [h_eq, h_fin_s, h_fin_t]
  omega
```

## Alternative: Explicit @ Call

If `haveI` triggers the same error, pass instances explicitly:

```lean
have h_fd_s : FiniteDimensional F s :=
  FiniteDimensional.of_finrank_pos (...)
have h_fd_t : FiniteDimensional F t :=
  FiniteDimensional.of_finrank_pos (...)
have h_eq := @Submodule.finrank_sup_add_finrank_inf_eq F V _ _ _ s t h_fd_s h_fd_t
```

## DO NOT Use

- `Submodule.rank_sup_add_rank_inf_eq` (Cardinal version — triggers Max Type)
- `FiniteDimensional.mk` (doesn't exist in this version)
- `Module.finite_of_finrank_pos` (same Max Type issue)
- `import Mathlib.LinearAlgebra.FiniteDimensional` (directory, not a module)
