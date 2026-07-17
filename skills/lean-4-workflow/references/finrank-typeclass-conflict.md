# FiniteDimensional typeclass conflict — REFUTED (2026-06-30 investigation)

## The "Max Type" error was a MISDIAGNOSIS

````lean
error: failed to synthesize instance of type class Max Type
````

A 2026-06-30 systematic investigation proved that the "Max Type" error
at `Submodule.finrank_sup_add_finrank_inf_eq` in Centering.lean line 1278
was NOT a universe/typeclass issue.  The actual root cause was a
**`LinearEquiv.finiteDimensional` direction error** at line 1271:

```lean
-- ❌ WRONG: h_equiv.symm.finiteDimensional
--    LinearEquiv.finiteDimensional direction is DOMAIN → CODOMAIN
--    h_equiv : F ≃ₗ[F] sId
--    h_equiv.symm.finiteDimensional needs FD(sId) to give FD(F) — backwards!
exact h_equiv.symm.finiteDimensional

-- ✅ CORRECT: h_equiv.finiteDimensional (FD(F) is trivially available)
haveI : FiniteDimensional F F := inferInstance
exact h_equiv.finiteDimensional this
```

Because the `FiniteDimensional F sId` instance failed to be provided (due to
the direction error), `Submodule.finrank_sup_add_finrank_inf_eq` could not
synthesize its `[FiniteDimensional K ↥s]` typeclass argument.  The resulting
error message — "failed to synthesize instance of type class Max Type" — was
a **generic typeclass-failure message**, not a specific universe issue.

**Correction**: `Submodule.finrank_sup_add_finrank_inf_eq` WORKS on Submodules
of `V F := ℕ → ℕ → ℕ → ℕ → F` when both `FiniteDimensional` instances are
correctly provided via `LinearEquiv.finiteDimensional` (not `.symm`).

## What DOES work (verified 2026-06-30)

- ✅ `FiniteDimensional.of_finrank_pos` (exists in mathlib, see `references/mathlib-finrank-sup-compatibility.md`)
- ✅ `bridge_equiv.symm.finiteDimensional` for kerΦ (bridge_equiv is kerΦ → ConstraintKernel)
- ✅ `h_equiv.finiteDimensional` for sId (when h_equiv is F → sId)
- ✅ `Submodule.finrank_sup_add_finrank_inf_eq` with correctly-provided FD instances
- ✅ `Module.finrank` on individual `Submodule F (V F)` values
- ✅ `finrank_span_singleton`

## Pitfall: `LinearEquiv.finiteDimensional` direction

`LinearEquiv.finiteDimensional (f : V ≃ₗ[K] V₂)` goes **domain → codomain**:

```lean
-- f : V ≃ₗ[K] V₂
-- f.finiteDimensional      : FiniteDimensional K V  → FiniteDimensional K V₂   (domain → codomain)
-- f.symm.finiteDimensional : FiniteDimensional K V₂ → FiniteDimensional K V    (codomain → domain)
```

**Diagnostic rule**: when you see `Max Type` on a finrank lemma, FIRST verify
that all `FiniteDimensional` instances are being provided in the correct
direction — before investigating universe issues.

## Original (retained for historical reference)

The rest of this document was written before the 2026-06-30 investigation
and has been superseded.  The `FiniteDimensional.of_finrank_pos` approach
(described in `mathlib-finrank-sup-compatibility.md`) should work; it simply
was never tested because the old proof was never actually executed against a
compiled environment — it existed only behind stale `.olean` cache.
