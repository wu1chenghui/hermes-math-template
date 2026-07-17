# finrank lower bound from a coeff-level independence lemma (across the bridge)

After the representation bridge is built (`kerΦ ≃ₗ ConstraintKernel`, see
`representation-bridge-finrank-carrier.md`), the dimension theorem splits into a
lower bound `finrank kerΦ ≥ N` and an upper bound `≤ N`. This file is the
**lower-bound assembly**: turning an *unbundled* coefficient-level independence
lemma (`∀ idx, (Σ scalars·basisfn idx) = 0 → all scalars = 0`) into a Mathlib
`LinearIndependent`, then into `N ≤ finrank`. Pure linear-algebra plumbing — no
new mathematics. The pattern is general to any "dim of a solution submodule"
formalization where the carrier is a `Submodule` of a Pi space `V = ι → F`.

## The three sub-steps (Lean-compilable order: do these BEFORE the upper bound)

**L1 — generator membership.** Each basis generator `coeff_X : V` must be shown
`∈ kerΦ`. Factor through one reusable core:
```
lemma mem_kerPhi_of (hn) (D : SemanticStruct) (hI : filter D.coeff) (hV : validSupp D.coeff) :
    D.coeff ∈ kerPhi hn := ⟨hI, hV, fun … => semantic_phi_zero D …⟩
```
The Φ=0 obligation is discharged by the existing `semanticStruct_phi_zero`
(here `halfDerivation_phi_zero`); `filter`/`validSupp` are per-generator
`split_ifs` facts. `(D.coeff)` is defeq to the raw `coeff_X`, so
`coeff_X_mem := mem_kerPhi_of hn D_X (…) (…)` typechecks directly.

**L2 — bundle into `LinearIndependent`.** This is the fiddly core.

* **Index encoding decision (matters a lot).** Index the family by
  `{k // k ∈ Finset.Icc a b} ⊕ Fin m`, NOT `Fin (b-a+1) ⊕ Fin m`. The
  `Icc`-subtype makes the sum reindex a single `Finset.sum_coe_sort` (no `+1`
  offset gymnastics with `Fin.sum_univ_eq_sum_range`/`Finset.sum_Icc_eq_sum_range`),
  AND supplies the membership bounds (`1 ≤ k`, `k ≤ b`) for free from
  `Finset.mem_Icc.mp p.2`. Card is still `(Icc).card + m` via `Finset.card_Icc`.
* **`![…]`-valued family + rfl helpers.** Define the family with `Sum.elim` and a
  `![g0,…,g4]` matrix for the finite block. Then prove `@[simp]` value lemmas
  `(genFamily hn (Sum.inr 0)).val = coeff_B := rfl` (and inl: `… (Sum.inl p) = coeff_A p.val := rfl`).
  `![…] k` reduces definitionally so these are all `rfl`; the main proof uses them
  to unfold genFamily values under `simp only`.
* **The proof spine:**
  ```
  rw [Fintype.linearIndependent_iff]; intro c hc
  -- STEP 1: kerΦ-sum  →  pointwise V equation
  have key1 : ∀ idx…, ∑ i, c i * ((genFamily hn i).val idx…) = 0 := by
    intro …
    have h := congrArg (fun x : kerPhi hn => x.val idx…) hc
    simpa only [Submodule.coe_sum, SetLike.val_smul, Finset.sum_apply,
      Pi.smul_apply, smul_eq_mul, ZeroMemClass.coe_zero, Pi.zero_apply] using h
  -- STEP 2-3: align with the unbundled lemma's hypothesis shape
  set cA : ℕ → F := fun k => if h : k ∈ Icc a b then c (Sum.inl ⟨k,h⟩) else 0 with hcA
  have hcaeq : ∀ p, cA p.val = c (Sum.inl p) := by
    intro p; simp only [hcA]; rw [dif_pos p.2, Subtype.coe_eta]
  have hsum : ∀ idx…, (∑ k ∈ Icc a b, cA k * basisA k idx…) + … = 0 := by
    intro …; have hk := key1 …
    rw [Fintype.sum_sum_type, Fin.sum_univ_five] at hk
    simp only [genFamily_inl_val, genFamily_inr0_val, …] at hk
    have hA : (∑ k ∈ Icc a b, cA k * basisA k …)
            = ∑ p : {k // k ∈ Icc a b}, c (Sum.inl p) * basisA p.val … := by
      rw [← Finset.sum_coe_sort (Icc a b) (fun k => cA k * basisA k …)]
      exact Finset.sum_congr rfl (fun p _ => by rw [hcaeq p])
    rw [hA]; linear_combination hk      -- linear_combination absorbs +-assoc/comm
  -- STEP 4-5
  obtain ⟨hAz, hB, …⟩ := indep_core hn cA (c (Sum.inr 0)) … hsum
  intro i; cases i with
  | inl p => have h0 := hAz p.val p.2; rwa [hcaeq p] at h0
  | inr j => fin_cases j <;> [exact hB; exact hC; …]
  ```
  `linear_combination hk` is the right closer for STEP 3: the goal and `hk` are the
  same equation up to `+`-associativity (`Fintype.sum_sum_type` gives
  `Asum + (Fin-sum)`, your stated goal is left-assoc) — `linear_combination`
  treats the `∑` as an atom and rebalances by ring. Do NOT try `exact hk`
  (assoc mismatch) or `linarith` (`F` is an unordered field).

**L3 — the finrank inequality.**
```
lemma finrank_kerPhi_ge (hn : 4 ≤ n) : N ≤ Module.finrank F (kerPhi (show 3 ≤ n by omega)) := by
  haveI : FiniteDimensional F (kerPhi (show 3 ≤ n by omega)) :=
    (bridge_equiv (show 3 ≤ n by omega)).symm.finiteDimensional   -- imported via the bridge
  have hcard := (genFamily_linearIndependent hn).fintype_card_le_finrank
  simp only [Fintype.card_sum, Fintype.card_coe, Fintype.card_fin, Nat.card_Icc] at hcard
  omega
```

## Pitfalls (all hit and resolved this session)

* **`(x : V)` ascription on a Submodule element fails to insert the coercion.**
  `(genFamily hn i : V F)` → "type mismatch: has type ↥(kerΦ) but expected V F".
  Use `(genFamily hn i).val` (the Subtype projection) instead — robust. Same for
  the `congrArg` lambda: `fun x => x.val idx…`, not `fun x => (↑x : V) idx…`.

* **`Fintype.card (SubtypeOrSum)` restated standalone fails instance synthesis.**
  Writing `have : Fintype.card ({k // k ∈ Icc a b} ⊕ Fin m) = N` triggers a fresh
  `Fintype (↥(Icc …) ⊕ Fin m)` synthesis that can fail. Instead `simp only
  [Fintype.card_sum, Fintype.card_coe, Fintype.card_fin, Nat.card_Icc] at hcard`
  — rewrite the EXISTING hypothesis that already carries the instance, then `omega`
  (treats `finrank` as an opaque ℕ atom: from `LHS ≤ finrank` and `LHS = N` it
  derives `N ≤ finrank`).

* **FiniteDimensional of a submodule of an INFINITE-dim ambient is NOT free.**
  `kerΦ ⊆ V = ℕ⁴→F` (infinite). `infer_instance` for `FiniteDimensional F (kerΦ)`
  FAILS. It must be transported from the finite side via
  `(bridge_equiv hn).symm.finiteDimensional`. (By contrast `ConstraintKernel ⊆`
  the finite `AdjData` IS auto-`FiniteDimensional`.) Confirm with a one-line
  `infer_instance` probe before relying on it (pin `F` with `(F := F)` or the
  probe stalls on `CharNeTwo ?m` — an elaboration-order artifact, not a real gap).

* **Stale partial-proof block from a prior session/turn.** When filling a long
  proof body left as `sorry`, READ the full surrounding region first. A prior
  partial attempt may sit below your `sorry`; a `sorry`-anchored `patch` can
  fuzzy-match the wrong spot and leave a duplicate block dangling AFTER the proof
  completes → "no goals" / dead-code errors. Delete the stale block explicitly,
  anchoring on a unique line (e.g. the differing indentation of the two `exact`s).

## Method discipline that worked
Skeleton-with-`sorry` first → `lean_goal` at the sorry to read the exact post-
`Fintype.linearIndependent_iff` state → fill ONE sub-step (`have key1`) → check
`lean_diagnostic_messages severity=error` → next sub-step. Each increment compiled
before the next; total lower bound (L1+L2+L3, ~22 lemmas) closed at 0 sorry/0 error.
