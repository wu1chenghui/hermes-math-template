# Representation Bridge — Classification → finrank assembly (Lean)

When a classification theorem (`dim Der/Hom/... = f(n)`) is **mathematically
closed on both bounds**, Lean work shifts from *proof phase* to
**integration / system-convergence phase**. You are no longer discovering
mathematics — you are gluing already-verified coefficient-level facts into one
linear-algebra spine. This file is the reusable playbook for that phase.

## Phase discipline (split actions into three classes; do not mix)

1. **Math layer** — FROZEN. No new lemmas, no collapse arguments, no new algebra.
2. **Lean layer** — integration only: type alignment, linear maps, isomorphism
   skeletons. Local 0-sorry files exist but the global import graph is not yet
   unified.
3. **Doc layer** — the ONLY real risk is *model drift* across AGENTS.md /
   ARCHITECTURE.md / paper. When a model is superseded (e.g. an old `dim=n`
   claim falsified into `dim=n+5`): **version-mark, never delete.** Prepend a
   `⚠️ DEPRECATED / SUPERSEDED` banner that points to the live source of truth.
   "Historically correct but obsolete" is the right framing for a frozen-but-wrong
   artifact — silently leaving it unmarked lets Lean and paper diverge into two
   self-consistent-but-inconsistent worlds.

## The carrier principle (decisive)

> **finrank / linear-independence / range must live on the object where the
> linear structure is NATIVE, not where the definition originates.**

A semantic structure (e.g. `structure HalfDerivation where coeff; half_leibniz : Prop`)
carries a `Prop` field and is **not naturally a Module**. Do NOT transport a
Module onto it via `Function.Injective` if your constraints are already stated at
the coefficient level — that adds a redundant transport layer and forces every
lower-bound / row-space lemma to be re-lifted.

Instead, make the **coefficient Pi-space** the ambient Module and cut the kernel
out as a Submodule:

```
SemanticStruct (origin)
      │ coeff embedding
      ▼
V := ℕ→ℕ→ℕ→ℕ→F          -- ambient Pi-module; Mathlib gives Module F V for free
      │ constraints (Φ = 0 ∧ filtered)
      ▼
kerΦ ⊆ V  (Submodule)    -- ★ the SOLE finrank carrier
      │ evAdj (evaluate at the finitely-many free coordinates)
      ▼
AdjData  (finite-dim Module)
      │ range
      ▼
ConstraintKernel := (evAdj).range
```

## The spine — dependency collapses to ONE injectivity lemma

```lean
def kerPhi (hn) : Submodule F (V F)                       -- finrank carrier
def evAdj  (hn) : kerPhi hn →ₗ[F] AdjData F n             -- evaluation morphism
def ConstraintKernel (hn) : Submodule F (AdjData F n) := LinearMap.range (evAdj hn)
theorem evAdj_injective (hn) : Function.Injective (evAdj hn)   -- ★ the one real lemma
noncomputable def bridge_equiv (hn) : kerPhi hn ≃ₗ[F] ConstraintKernel hn :=
  LinearEquiv.ofInjective (evAdj hn) (evAdj_injective hn)      -- nearly free once injective
```

Everything downstream (`finrank kerΦ` ⇒ `dim = f(n)`) reduces to:
- **Lower bound**: an `indep_core`-style coordinate-injectivity statement (a zero
  combination of the generators, evaluated at each generator's identifying index,
  forces all scalars to vanish) ⇒ `finrank ≥ k`.
- **Upper bound** = `evAdj_injective` + a rank bound on `ConstraintKernel`
  (Generation Closure / Stratification, already paper-closed) ⇒ `finrank ≤ k`.
- `evAdj_injective` itself is pure glue: its engines (width-stability lemmas killing
  the off-free coordinates, plus the filtered predicate killing non-target entries)
  are already proven 0-sorry. There is no new mathematics in the bridge.

## Machine-safe skeleton idiom (compile first, fill later)

Write the whole file as a **compiling skeleton with `sorry` placeholders**, verify
it elaborates (`lake env lean <file>` → only `declaration uses sorry` warnings, zero
errors), THEN fill one `sorry` at a time, recompiling between each. Do NOT dump a
full proof file in one shot.

- **Submodule** via anonymous-constructor `where`; use **term-mode `sorry`** for the
  three obligations (not tactic blocks — avoids implicit-binder fuss):
  ```lean
  def kerPhi (hn) : Submodule F (V F) where
    carrier := { c | IsKerPhi hn c }
    add_mem' := sorry
    zero_mem' := sorry
    smul_mem' := sorry
  ```
- **LinearMap** via `where` with `toFun` / `map_add'` / `map_smul'` (props = `sorry`).
- **Submodule element → ambient**: use `T.val` (an element of `↥M` is a subtype).
- **Onto-range equiv**: `LinearEquiv.ofInjective f hf` gives `domain ≃ₗ range f` —
  so `bridge_equiv` is free once `evAdj_injective` lands. Mark it `noncomputable`.
- Submodule-closure `add_mem`/`smul_mem` follow from filtered-predicate closure plus
  `Phi_add`/`Phi_smul`, which are one-liners off the project's
  `bracketSource_add/smul` + `bracketIdentity_add/smul` linearity lemmas.

## Build-order gotcha (see SKILL.md Pitfalls)

The bridge file imports project modules that were validated only by `lake env lean`
and therefore have **no `.olean`**. `lake build <Module.Name>` them first, otherwise
the import fails with `object file '...does not exist'`. And rely on `import
Mathlib.Tactic` for `Submodule`/`LinearMap`/`LinearEquiv`/`range` — adding a fresh
`import Mathlib.LinearAlgebra.Basic` fails the same way (its `.olean` was never built
in this project's closure).
