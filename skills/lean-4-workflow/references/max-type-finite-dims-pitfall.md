# Max Type Pitfall: FiniteDimensional on ℕ⁴ → F — REFUTED (2026-06-30)

> ⚠️ **THE CLAIMS IN THIS DOCUMENT HAVE BEEN REFUTED.**  The 2026-06-30 systematic
> investigation proved that the "Max Type" error was a MISDIAGNOSIS.  The actual
> root cause was a `LinearEquiv.finiteDimensional` direction error (see below).
> This document is retained for historical reference only.

## What was actually wrong

The "Max Type" error at `Submodule.finrank_sup_add_finrank_inf_eq` in Centering.lean
line 1278 was caused by a missing `FiniteDimensional` instance — NOT by a universe
issue.  The `FiniteDimensional F sId` instance failed to be provided because
`h_equiv.symm.finiteDimensional` was used when `h_equiv.finiteDimensional` was needed:

```lean
-- h_equiv : F ≃ₗ[F] sId

-- ❌ WRONG in Centering.lean line 1271:
exact h_equiv.symm.finiteDimensional
-- This needs FiniteDimensional F sId to produce FiniteDimensional F F
-- But we're TRYING to provide FiniteDimensional F sId!

-- ✅ CORRECT:
haveI : FiniteDimensional F F := inferInstance
exact h_equiv.finiteDimensional this
-- This needs FiniteDimensional F F (trivially available) to produce FiniteDimensional F sId
```

Once the correct direction is used, `Submodule.finrank_sup_add_finrank_inf_eq`
synthesizes its `[FiniteDimensional K s] [FiniteDimensional K t]` typeclass
arguments and works correctly.

## What this means for the approaches listed below

The "❌ Failed" approaches listed in the original document were never actually
tested with CORRECTLY PROVIDED `FiniteDimensional` instances.  They were tested
with the wrong-direction `LinearEquiv.finiteDimensional` call, which made it
appear that ALL approaches failed.  The correct conclusion is:

- ✅ `Submodule.finrank_sup_add_finrank_inf_eq` WORKS on Submodules of `V F`
- ✅ `LinearEquiv.finiteDimensional` pattern WORKS for providing FD instances
- ✅ `FiniteDimensional.of_finrank_pos` EXISTS and should work

See `references/finrank-typeclass-conflict.md` and
`references/mathlib-finrank-sup-compatibility.md` for the corrected guidance.

---

## Original document (REFUTED — retained for historical reference only)
