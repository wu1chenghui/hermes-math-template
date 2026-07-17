# Representation Bridge & finrank Carrier (coeff-space → finite model)

How to formalize **"dim of a solution space = N"** when solutions are coefficient
tables `ℕ→…→F` satisfying linear constraints (½-derivations, cocycles, ker of a
linear operator Φ). Pattern verified building `RepresentationBridge.lean` for the
1/2-derivation classification (kerΦ ≃ ConstraintKernel ⊆ AdjData, dim n+5).

## The architecture (the "spine")

```
HalfDerivation (semantic origin)
   │ coeff embedding
V = ℕ→ℕ→ℕ→ℕ→F     ambient Pi-module — linear structure is NATIVE here (Pi.module)
   │ cut by predicates
kerΦ ⊆ V           ★ Submodule — the UNIQUE finrank carrier
   │ evAdj : evaluate at the finite "free coordinates" (e.g. adjacent sources × channels)
AdjData            finite-dim coordinate model (a product of Fin→F)
   │ range
ConstraintKernel := (evAdj).range
```

- Keep the ambient = the **coefficient Pi-space** (`ℕ→…→F`). Mathlib's `Pi.module`
  gives it a Module for free, and ALL your frozen toolbox lemmas (the operator,
  the bracket/identity lemmas, width/stability lemmas) are already stated over it.
  Do NOT change the ambient to a finite type — that severs the frozen API and bakes
  the reduction theorem into the type (see "carrier choice" below).
- `kerΦ : Submodule F V` is the **only place `finrank` lives**.
- `evAdj : kerΦ →ₗ[F] AdjData` evaluates each element at the finite set of free
  coordinates. `ConstraintKernel := LinearMap.range (evAdj …)`.
- `bridge_equiv := LinearEquiv.ofInjective (evAdj …) (evAdj_injective …)` — once
  injectivity is proven, the iso `kerΦ ≃ₗ ConstraintKernel` is essentially free.
- `dim` then reduces to a pure `finrank` computation on the finite model: lower
  bound from an explicit independent generating set, upper bound from injectivity
  + the constraint structure on `AdjData`.

## ★ THE central pitfall: a function-space carrier OVER-REPRESENTS the object

`V = ℕ⁴→F` has "junk" degrees of freedom at **illegal indices** (e.g. `i ≥ j`, or
out of `[1,n]`). If you cut the kernel only by
`{ operator Φ = 0 (validity-guarded) ∧ target-filter }`, the kernel still
**contains illegal-index junk** and is therefore **infinite-dimensional**, and the
evaluation morphism is **NOT injective**.

Counterexample template (this killed an early carrier design):
```
g i j u v := if (i,j,u,v) = (illegal-source, legal-target-in-image-ideal) then 1 else 0
```
`g` is target-filtered (only nonzero target is in the allowed set), satisfies the
guarded operator condition vacuously (all *legal*-index evaluations of Φ never
reference the illegal source), is `≠ 0`, and has zero image under `evAdj` — so
`evAdj g = evAdj 0` with `g ≠ 0`. Injectivity is false.

**Why:** target-filters constrain only the target; a *validity-guarded* operator
condition says nothing at illegal indices. Neither pins illegal-source coordinates.

### Fix — add a "valid-supported" cutting condition to the CARRIER predicate

```lean
def IsValidSupp (c : V F) : Prop :=
  ∀ i j u v, ¬(1 ≤ i ∧ i < j ∧ j ≤ n ∧ 1 ≤ u ∧ u < v ∧ v ≤ n) → c i j u v = 0

def IsKerPhi hn c := I_filtered c hn ∧ IsValidSupp c ∧ (∀ legal idx, Phi c … = 0)
```

This selects the genuine object out of the over-representation. It is a
**subspace** (closed under +/•/0 — proofs are trivial, same shape as the
target-filter closure), so finrank works and the junk counterexample is excluded.

Slogan: **the bug is "free completion too big", not "predicate too weak" in the
wrong place — shrink the SOLUTION SUBSPACE inside a fixed ambient, by a predicate.**

### Carrier-choice decision: flat predicate-cut Submodule vs subtype-ambient

Two ways to add valid-support give the SAME submodule of `ℕ⁴→F`:
- **flat (USE THIS):** ambient stays `ℕ⁴→F`; valid-support is one conjunct of the
  kernel predicate. One `.val` coercion; frozen toolbox applies directly via `.val`;
  `evAdj/AdjData/range/equiv` unchanged.
- **subtype-ambient (avoid):** ambient `:= {f : ℕ⁴→F // ValidSupp f}`. Then kerΦ is a
  "submodule of a submodule" → **double `.val`**, awkward Module/Submodule instances,
  must re-encode `evAdj` domain. Zero mathematical gain.

## The coeff → structure witness lift (frozen lemmas want a `structure`, not raw coeff)

Frozen toolbox lemmas (e.g. `width_stability_*`, anything over `coeffOf D`) are
often phrased over a **structure** `D : HalfDerivation` and its bounds-checking
accessor `coeffOf D`, NOT over a raw `c : ℕ⁴→F`. To use them on a kernel element
`T` (whose `T.val` is the coeff), build an **ephemeral local witness** inside the
proof:
```lean
have D : HalfDerivation F n := ⟨T.val, by
  intro i j k l u v hi hij hjn hk hkl hln hu huv hvn
  -- kernel's Φ=0 conjunct gives Phi T.val … = 0; Phi := 2*bracketSource − bracketIdentity
  exact sub_eq_zero.mp (hΦ i j k l u v hi hij hjn hk hkl hln hu huv hvn)⟩
-- now: width_stability_a D … : coeffOf D … = 0 ; bridge back with coeffOf_f to T.val
```
This is **acyclic** (the structure-API module does not import the bridge) and the
witness must **never appear in a signature** — it is a proof artifact only.
Document it as such (a Lean reviewer will misread the DAG otherwise).

## Concrete Lean 4 gotchas hit this session

- **Implicit `{n}` not inferable from a `V F = ℕ⁴→F` argument** (the arg type does
  not mention `n`). Symptom: `don't know how to synthesize implicit argument n`,
  metavariable `?m` in the goal header. Fix: pin via type ascription on one side of
  the equation, `(evAdjFun (a+b) : AdjData F n) = …`, or `IsValidSupp (n := n) c`.
- **`injective_iff_map_eq_zero`** (NO `LinearMap.` prefix — that name doesn't exist).
  `rw [injective_iff_map_eq_zero]` turns `Function.Injective ⇑f` into
  `∀ a, f a = 0 → a = 0` for any LinearMap (it's an `AddMonoidHomClass`).
- **Pi/Prod evaluation respects +/•** by `rfl`: `evAdjFun (a+b) = evAdjFun a + evAdjFun b := rfl`
  and the `smul` analogue both close by `rfl` (structural defeq through `Pi`/`Prod`).
- **Building a `Submodule` via `where`** for a skeleton: `add_mem'/zero_mem'/smul_mem' := sorry`
  (term-mode `sorry`). With an `n`-conjunct carrier predicate, destructure membership
  with `rintro a b ⟨h1,h2,h3⟩ …` and split the goal with `refine ⟨?_,?_,?_⟩`.
- **`T = 0` for a Submodule element**: `apply Subtype.ext; funext i j u v; show (T : V F) i j u v = 0`.
- **Φ-linearity** (`Phi_add/Phi_smul/Phi_zero`) for the kernel closure: `unfold Phi
  bracketSource bracketIdentity bracketLeft bracketRight; simp only [Pi.add_apply /
  Pi.smul_apply, smul_eq_mul / Pi.zero_apply]; split_ifs <;> ring`.

## Build / import gotchas

- **Modules verified only via `lake env lean <f>` have NO `.olean`.** If a module is
  not in the package entry-point's import tree (`E.lean`), `lake build` never built it,
  so a new file that `import`s it fails with `object file … does not exist`. Fix:
  `lake build <Fully.Qualified.Module>` first (this also builds its transitive deps),
  then compile the new file.
- **Don't `import Mathlib.LinearAlgebra.Basic`** in a project whose files only
  `import Mathlib.Tactic`: that `.olean` isn't in the build closure (`object file …
  does not exist`). `Mathlib.Tactic` already transitively provides `Submodule`,
  `LinearMap`, `LinearEquiv`, `LinearMap.range`, `Function.Injective` lemmas. Add a
  specific Mathlib module only if a symbol is genuinely missing, and `lake build` it.

## Workflow: reconcile a math-level architecture sketch against the FROZEN code first

When a collaborator hands you a math-level carrier/proof architecture, its
Lean-level details are usually approximate. **Before writing**, verify against the
actual signatures:
- Which space does each lemma live on? ("These are V-level theorems" is often false —
  they take a `structure`/`coeffOf`, forcing the witness lift above.)
- Is the operator condition **guarded** (only legal indices) or unguarded? They are
  different objects; the guarded one matches the axiom and contains the basis, the
  unguarded one may exclude the basis.
- Apply the SAME rigor to the collaborator's proposals: if a proposed carrier or
  "no-junk" lemma is unsound, reject it with a concrete counterexample (the `g`
  template) rather than trying to prove a false statement.

## Finrank assembly layer (the D-step bookkeeping AFTER `bridge_equiv` exists)

Once `bridge_equiv : kerΦ ≃ₗ ConstraintKernel` is built, computing `finrank = N`
splits into a few facts. **All of these were machine-verified** (probe in
`DimensionTheorem.lean`) — do NOT re-derive them by hand, and do NOT assume (as a
paper sketch may imply) that finite-dimensionality requires the upper bound:

- **finite side is FREE.** `ConstraintKernel ⊆ AdjData` where `AdjData = (Fin k→F)ⁿ`
  is finite-dim, so `FiniteDimensional F (ConstraintKernel hn)` resolves by
  `infer_instance` (submodule of a f.d. space). `finrank (AdjData F n)` is also
  free: e.g. `(Fin(n-1)→F)³` has `finrank = (n-1)+((n-1)+(n-1))` by
  `simp [AdjData, Module.finrank_prod]` (`Module.finrank_pi` is *unused* here).
- **infinite side is NOT free, but is a 1-liner.** `kerΦ ⊆ V = ℕ⁴→F` is an
  *infinite-dim* ambient, so `FiniteDimensional F (kerΦ hn)` FAILS `infer_instance`
  (`synthInstanceFailed`). Transport it: `(bridge_equiv hn).symm.finiteDimensional`.
- **`finrank` transfer is unconditional 1-line:** `finrank kerΦ = finrank
  ConstraintKernel := (bridge_equiv hn).finrank_eq`. (`LinearEquiv.finrank_eq`
  needs no f.d. hypothesis.) So "finrank kerΦ = N" ⟺ "finrank ConstraintKernel = N".
- **★ CharNeTwo-metavariable `infer_instance`/finrank stuck error.** Probing a def
  that carries `[CharNeTwo F]` in its signature (kerΦ/ConstraintKernel/bridge_equiv)
  with `example : … := by infer_instance` or a `.finrank_eq` term can error with
  `typeclass instance problem is stuck: CharNeTwo ?m.6 … arguments are metavariables`.
  This is an **elaboration-order artifact, NOT a math gap** — `F` wasn't pinned before
  instance search. Fix: supply `(F := F)` (and `(n := n)` if needed) explicitly. Defs
  whose type doesn't mention `[CharNeTwo F]` (e.g. `AdjData`) don't hit this.

### Where the only real obligation lives — and the recommended route

For `finrank ConstraintKernel = N`, the **bottleneck is the UPPER bound**
(`≤ N`). The lower bound, the finrank API, and the f.d. transport are all
bookkeeping (above). Decision table for "what's the bottleneck":
finrank API → no (works on all three carriers); f.d. transport → no (free/1-liner);
representation bridge → no (done by `bridge_equiv`); **rank / Stratification → YES.**

- **Lower bound (`≥ N`) is plumbing**, but note the SHAPE gap: an `indep_core`-style
  lemma is usually **unbundled** — `(h : ∀ idx, Σ cₖ·genₖ idx = 0) → all cₖ = 0`
  (pointwise-zero of a V-valued combination), NOT Mathlib `LinearIndependent`.
  Bridging needs: (a) each generator `∈ kerΦ` (the half-Leibniz/Φ=0 conjunct comes
  from the `halfDeriv_*` witnesses; the I_filtered/valid-supp conjuncts are by
  construction but must be *stated*); (b) `unbundled → LinearIndependent F (g : Fin N
  → kerΦ)` via `funext`/Pi-zero; (c) `LinearIndependent + FiniteDimensional →
  N ≤ finrank`. ~4–6 mechanical lemmas.
- **Upper bound (`≤ N`) — two sub-routes.** (α) build the constraint matrix and prove
  `Matrix.rank = …`: Mathlib `Matrix.rank` ergonomics are painful, needs explicit
  pivot/block bookkeeping — **avoid**. (β, RECOMMENDED) prove `kerΦ ≤ span{the N
  generators}` (every element is a combination of generators); combined with the
  lower-bound independence this yields a `Basis` of exactly `N` elements and
  `finrank = N` in ONE shot via `finrank_eq_card_basis` — **both bounds at once**,
  reusing the width/stability lemmas instead of building matrix-rank infra. The
  spanning-decomposition lemma IS the genuine remaining math content (the rank fact,
  repackaged); everything else is plumbing.

### Audit-probe technique (map "free vs needs-a-lemma" before writing proofs)

Before committing to a final-assembly proof, do a **dependency audit**: create the
skeleton file (imports only, confirm `lake env lean … = 0 error` = "D0"), then
temporarily add `#check`/`example : … := by infer_instance`/`example : eq := lemma`
probes, compile via `lake env lean`, and read which SUCCEED vs ERROR. `#check` prints
the elaborated type (success); a failing `infer_instance` prints `synthInstanceFailed`
(documents a real gap). Then **restore the clean skeleton**. This tells you exactly
which facts are free and which need a lemma — so you know "how many lemmas to the
end" instead of guessing. (Sibling of the unsolved-goals goal-probe; this one maps
instances/finrank rather than goal text.)
