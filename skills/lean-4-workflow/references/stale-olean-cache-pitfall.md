# Stale Olean Cache Pitfall

## The Problem

`lake build` can report success ("2967 jobs, 0 errors") even when a module
has real compile errors. This happens because:

1. The `.olean` file for a module was compiled from an EARLIER version of the
   source code (before the buggy code was added, or from an intermediate state).
2. Subsequent `lake build <Module>` calls see the existing `.olean` and skip
   recompilation.
3. The buggy code is NEVER actually compiled — it's invisible to the build system.

## Concrete Example (2026-06-29, Centering.lean)

AGENTS.md checkpoint "2026-06-29 b" claimed:
> lake build = 2967 jobs, 0 error
> D4 COMPLETE: finrank_halfDer = n+5 PROVED

Reality after `lake clean` + full `lake build` (2026-06-29):
- `HalfDerivation` lacked `AddCommMonoid` and `Module` instances needed by `coeffOf_map`
- `coeffOf_kerPhi_lift` had `.symm` direction errors (6 branches)
- `Submodule.disjoint_iff`, `Submodule.finrank_sup_eq_of_disjoint` don't exist in mathlib v4.31
- `finiteDimensional_of_finrank` doesn't exist — must use `FiniteDimensional.of_finrank_pos`
- **~18 compile errors total**, not just finrank issues:
  - Line 544: `Unknown identifier 'i'` in `nonadjacent_target_n_zero` (likely `simp` behavior change in v4.31)
  - Lines 1124-1134: `Unknown identifier 'i'/'hi'/'hij'/'hjn'/'hu'/'huv'` cascade in `coeffOf_map` section (HalfDerivation instance issues)
  - Lines 1233/1238: `Max (Type u)` on finrank sup lemma (fundamental mathlib v4.31 issue, no known workaround)
  - Line 1241: `assumption` cascade error
- `FinrankHelper.lean` (the isolation file) itself triggers `Max (Type u)` even with `universe u`, explicit `M : Type u`, and minimal imports
- **The "D4 COMPLETE" status was entirely an artifact of stale olean cache.**

The `coeffOf_map` definition uses `→ₗ[F]` which requires `AddCommMonoid` on
the domain. Since `HalfDerivation` has no such instance, `coeffOf_map` could
NEVER have compiled. The olean cache was from a time before this code was added.

## Detection

If a theorem's proof references functions that require typeclass instances not
present in the source, the proof was never compiled:

```lean
def coeffOf_map : HalfDerivation F n →ₗ[F] V F where ...
-- Requires: AddCommMonoid (HalfDerivation F n), Module F (HalfDerivation F n)
-- If HalfDerivation has no such instances → this could never compile
```

## Remedy

1. Run `lake clean` then `lake build` to verify from scratch (slow — rebuilds mathlib)
2. Or delete just the suspect module's `.olean` and rebuild:
   ```bash
   rm .lake/build/lib/lean/E/Classification/Centering.olean
   lake build E.Classification.Centering
   ```
3. Before claiming "build-verified" status in AGENTS.md, always run at least #2.
