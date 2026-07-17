# Spanning / Upper-Bound: the Interface-Prelude Pattern

Context: proving a `finrank carrier ≤ N` upper bound by showing `carrier ≤ span{generators}`,
where the carrier is a coefficient Pi-space `V F = ℕ→ℕ→ℕ→ℕ→F` `Submodule` (`kerΦ`) and the
evaluation morphism `evAdj : carrier →ₗ AdjData` lands in a finite coordinate model
(`AdjData = (Fin (n-1) → F)³`, channels `.1`/`.2.1`/`.2.2`). Sits on top of the
representation-bridge / finrank references in this same directory.

## Methodology: write the interface-prelude lemma FIRST

Do NOT write the full `spanning_kerPhi` theorem in one shot. The real risk in this layer is NOT
combinatorial explosion (already discharged by the constraint-collapse / generator-coordinate /
normal-form lemmas) — it is **type-interface mismatch** between:
  - `T : carrier` (Submodule element; `T.val : V F`, `T.property : IsKerPhi …`)
  - `evAdj T : AdjData` (finite coordinate tuple)
  - the AdjData-level normal form (`a_channel_normal_form` / `c_channel_normal_form`)
  - the constructed-S output (`∃ S, evAdj S = …` over a raw `V F` term)
  - the final `T ∈ Submodule.span {generators}` goal

Write a minimal **interface-prelude** that threads `T` through the pipeline with NO `span` /
`Submodule.span` / `genFamily` / `finrank`:
```
lemma evAdj_surj_on_normal_forms (hn : 4 ≤ n) (T : carrier) :
    ∃ S : carrier, evAdj S = evAdj T
```
If this compiles 0-error / 0-sorry / axiom-clean, the full `spanning_kerPhi` collapses to
`obtain ⟨S, hS⟩ := prelude; have : S = T := Subtype.ext (evAdj_injective … ); rw[←this]; <S ∈ span>`.
The prelude surfaces ~80 % of the type-mismatch problems early, in isolation. This is the
user-ratified "Gate C" discipline: prove the interface bridge before the packaging theorem.

## Proof spine of the prelude
1. `obtain ⟨hI, hV, hΦ⟩ := T.property` (membership = I_filtered ∧ valid-supp ∧ guarded-Φ).
2. Read the construction parameters off the finite-model coordinates with `set … with`:
   `set a0 := (evAdjFun (T : V F) : AdjData F n).1 ⟨0,_⟩ with ha0`, …, `set bs := (…).2.1`.
3. Build `S_val : V F` as the explicit generator combination
   (`fun i j u v => ∑ p, bs p * coeff_A (p.val+1) … + c0 * coeff_B … + …`).
4. `S_val ∈ carrier` via Submodule closure (gotcha 6).
5. Interior constraints: instantiate `hΦ` at each per-row source, feed `interior_a/c_zero`.
6. Coupling (`a₁ + c_{n-1} = 0`): instantiate `hΦ` at the coupling source, gives the value of the
   ONE pinned coordinate the constructor hard-codes (here `c@(n-2) = -a0`). Skipping this is the
   classic "constructor pins a coordinate you must supply via coupling" trap.
7. Channel-by-channel equality: a via the normal-form lemma (defeq), b directly, c via normal-form
   + the coupling rewrite.

## The gotchas that each cost an iteration

1. **Implicit `n` becomes a metavar when the carrier type doesn't mention it.** `V F = ℕ→ℕ→ℕ→ℕ→F`
   carries no `n`, so `evAdjFun (T : V F)` leaves `n` unsolved → a Fin literal `⟨n-3, by omega⟩`
   elaborates against `Fin (?n - 1)` and `omega` fails ("counterexample n ≥ 4" / `b := ↑?m`).
   FIX: ascribe the result type to pin n: `(evAdjFun (T : V F) : AdjData F n).1 ⟨n-3, by omega⟩`.
   Inside a `have`/lemma whose statement already binds `p : Fin (n-1)`, the binder pins n — no
   ascription needed there. The metavar bites only standalone coordinate literals.

2. **`set x := e with h` makes a LET-binding (defeq to `e` by zeta), not an opaque fvar.** So
   defining parameters AS the finite-model coordinates lets the constructed normal form close by
   pure `rfl`: the a-channel value built from `a0 := (…).1 ⟨0⟩` is defeq to the normal-form
   lemma's ⟨0⟩-slot (literally `(…).1 ⟨0⟩`). Do NOT restructure to avoid "set opacity" — set vars
   are defeq to their bodies; the real blocker is always gotcha 1, not set.

3. **Nat subtraction is NOT defeq.** `(n-2)+1` ≠ `n-1` definitionally for variable `n`.
   `congr`/`rfl` leave an unprovable goal (`omega: counterexample`). FIX:
   `rw [show (n-2)+1 = n-1 from by omega, show (n-2)+2 = n from by omega]`.

4. **`linarith` does not work over a `Field`.** For `hcoup : a + b = 0 ⊢ b = -a` use
   `linear_combination hcoup`.

5. **Dirac-delta finite sum over `Fin`.** Goal `bs p = ∑ x, if ↑p = ↑x then bs x else 0`
   (condition via `Fin.val`, then-branch a function of the bound var): close with
   `simp only [Fin.val_inj, Finset.sum_ite_eq, Finset.mem_univ, if_true]`. Do NOT use
   `Finset.sum_ite_eq'` — wrong orientation; its simp args come back "unused" and the goal is left
   open (this is exactly the dead tactic a prior session left as the single residual error).
   `Fin.val_inj` collapses `↑p = ↑x` → `p = x`, then `sum_ite_eq` evaluates the delta.

6. **Submodule closure of a pointwise-defined combination.** To prove a `fun i j u v => Σ … + …`
   term `∈ carrier`, first rewrite it to a genuine Submodule combination:
   `funext i j u v; simp only [Pi.add_apply, Pi.smul_apply, Finset.sum_apply, smul_eq_mul]`, then
   `refine add_mem (add_mem … ?_) ?_` and discharge each leaf with
   `Submodule.sum_mem _ (fun p _ => Submodule.smul_mem _ _ (coeff_A_mem …))` /
   `Submodule.smul_mem _ _ (coeff_X_mem …)`. The per-generator `coeff_X_mem` lemmas already prove
   membership; closure is mechanical (`add_mem` nesting must mirror the term's left-assoc `+`).

7. **`show` that CHANGES the goal trips the style linter — use `change`.** When you intentionally
   rewrite the goal to a defeq form (`evAdj _ ⟨S_val,_⟩ = evAdj _ T` → `evAdjFun S_val = evAdjFun ↑T`,
   or `(evAdjFun ↑T).1 p` → `↑T (p+1)(p+2) 1 (n-1)`), use `change`, not `show`.

8. **Proof-irrelevance closes `nf = nf` that `rw`'s syntactic auto-rfl left open.** After a
   `rw [normal_form …]` the two sides can be structurally-identical `if`-lambdas differing only in
   `Fin` membership proofs. Let the let-binding (gotcha 2) make them defeq so the branch closes, or
   end with `exact h.symm` (elaboration uses defeq), not a bare `rfl` chained after the `rw`.

## Off-file verification workflow (zero-rework before落盘)
Prove the WHOLE lemma in `lean_run_code` first — `import <ProjectModule>` + `open …` +
`example (hn …) (T …) : … := by <proof>`. Iterate there (no file mutation) until 0-error. Use
`lean_multi_attempt` on a single stuck line to pick the closing tactic (it returns `goals: []` for
each candidate that closes). Only then `patch` it into the file. This landed a ~90-line lemma with
zero on-file rework. When porting the `example` into the file:
  - drop the local `variable` block (the file already has one in-namespace),
  - fully-qualify names for namespaces the file doesn't `open` (e.g. `DimensionTheorem.coeff_A_mem`),
  - convert goal-changing `show` → `change` (gotcha 7).
Then confirm with the three-source cross-check the project uses: LSP `lean_diagnostic_messages`
(0 error) + `lake build <Module>` (.olean regenerated) + `lean_verify` (axioms = propext /
Classical.choice / Quot.sound, no `sorryAx`).
