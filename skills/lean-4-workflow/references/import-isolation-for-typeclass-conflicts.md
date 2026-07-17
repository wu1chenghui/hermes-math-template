# Import Isolation for Typeclass Conflicts

**Problem**: A mathlib lemma fails with `Max Type` or other typeclass synthesis errors
when called from a file with many imports (e.g., Centering.lean with 12+ imports),
but the SAME lemma works in a standalone file with minimal imports.

**Root cause**: Complex import chains can create universe-level conflicts or
conflicting typeclass instances that prevent `FiniteDimensional`, `Module.Finite`,
`LinearEquiv.ofBijective`, and other lemmas from synthesizing for types like
`Submodule F (ℕ → ℕ → ℕ → ℕ → F)`.

**Fix**: Create a SEPARATE helper file with MINIMAL imports that proves the needed
lemma, then import that file into the main file.

## Recipe

1. Create a new `.lean` file (e.g., `E/Classification/FinrankHelper.lean`).

2. Import ONLY what's needed (typically `Mathlib.Tactic` + 1-2 mathlib modules):
   ```lean
   import Mathlib.Tactic
   import Mathlib.LinearAlgebra.FiniteDimensional.Basic
   import Mathlib.LinearAlgebra.FiniteDimensional.Lemmas
   ```

3. Define the lemma with explicit type annotations for the problematic type:
   ```lean
   lemma finrank_sup_disjoint {s t : Submodule F (ℕ → ℕ → ℕ → ℕ → F)}
       (h_disjoint : Disjoint s t) (h_fin_s : Module.finrank F s = a)
       (h_fin_t : Module.finrank F t = b) : Module.finrank F (s ⊔ t) = a + b := by
     ...
   ```

4. Use `LinearEquiv` to prove the equality (avoids `FiniteDimensional` typeclass):
   ```lean
   -- Build a LinearEquiv: (s ⊔ t) ≃ₗ[F] (s × t) using the disjointness condition
   noncomputable def supProdEquiv (h_disjoint : Disjoint s t) : (s ⊔ t) ≃ₗ[F] (s × t) :=
     LinearEquiv.ofBijective (sum_map) (injective_proof, surjective_proof)
   
   -- Then finrank equality follows from LinearEquiv.finrank_eq (no FiniteDimensional needed)
   have h_equiv := supProdEquiv ... h_disjoint
   rw [← (Module.finrank_prod ...), h_equiv.finrank_eq, h_fin_s, h_fin_t]
   ```

5. Import the helper file into the main file (remove the problematic direct imports):
   ```lean
   -- In Centering.lean, replace:
   -- import Mathlib.LinearAlgebra.FiniteDimensional.Basic   ← REMOVE
   -- import Mathlib.LinearAlgebra.FiniteDimensional.Lemmas  ← REMOVE
   -- With:
   import E.Classification.FinrankHelper  ← single clean import
   ```

6. Use the lemma with one line:
   ```lean
   have h_fin := finrank_sup_disjoint h_disjoint h_finrank_sId h_finrank_ker
   ```

## Fallback: `import Mathlib`

If even the minimal import approach fails (the `Max Type` issue persists),
try `import Mathlib` in the helper file. This forces the full mathlib environment
which sometimes resolves universe issues. The downside: first-time compilation
takes 30+ minutes (4000+ modules). Subsequent builds are instant (cached).

```lean
import Mathlib  -- nuclear option, but reliable
```

## Key Insight

The `LinearEquiv.finrank_eq` lemma works WITHOUT `FiniteDimensional` or
`Module.Finite` instances. It's a structural equality: isomorphic modules
have the same `Module.finrank` (which is `0` if infinite-dimensional).
This makes it the most robust approach when typeclass issues block
`FiniteDimensional.of_finrank_pos` or `Submodule.finrank_sup_add_finrank_inf_eq`.
