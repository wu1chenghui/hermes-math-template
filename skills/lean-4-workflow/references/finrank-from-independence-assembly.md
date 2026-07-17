# finrank lower bound from a bespoke unbundled independence lemma

The DimensionTheorem assembly layer that sits ON TOP of the representation bridge
(`references/representation-bridge-finrank-carrier.md`). Goal: turn a hand-rolled
**unbundled** independence lemma into `finrank kerΦ ≥ N`, where `kerΦ` is a
`Submodule F (V F)` and `V F = ℕ→ℕ→ℕ→ℕ→F` is an **infinite-dimensional** Pi-space.
Validated end-to-end (N = n+4) with 0 sorry, 2026-06-23.

Three sub-steps (do them in this Lean-compilable order, each compiling before the next):

## L1 — generator membership in the Submodule

`kerΦ` membership = `IsKerPhi` = `I_filtered ∧ IsValidSupp ∧ (Φ=0 guarded)`.
Build ONE reusable core, then feed each generator through it:

```lean
lemma mem_kerPhi_of (hn) (D : HalfDerivation F n)
    (hI : I_filtered D.coeff hn) (hV : IsValidSupp (n := n) D.coeff) :
    D.coeff ∈ kerPhi (F := F) hn := by
  refine ⟨hI, hV, ?_⟩          -- membership reduces to the And by defeq
  intro i j p q u v h1 h2 h3 h4 h5 h6 h7 h8 h9
  exact halfDerivation_phi_zero D i j p q u v h1 h2 h3 h4 h5 h6 h7 h8 h9
```

`(halfDeriv_X i).coeff` is **defeq** to `coeff_X i`, so `coeff_X_mem := mem_kerPhi_of hn (halfDeriv_X i) hIf hVs` typechecks directly.

Support lemmas (sparse `if`-coeff). Single-if (A,B,E):
```lean
-- I_filtered
intro a b u v hne; simp only [coeff_X] at hne
split_ifs at hne with h
· obtain ⟨_, _, hu, hv⟩ := h; subst hu; subst hv; simp [I_target_set]
· exact absurd rfl hne
-- IsValidSupp
intro a b u v hnv; simp only [coeff_X]; rw [if_neg]
rintro ⟨ha, hb, hu, hv⟩; subst ha; subst hb; subst hu; subst hv
exact hnv ⟨by omega, by omega, by omega, by omega, by omega, by omega⟩
```
Double-if (C,D,F): use `split_ifs ... with h1 h2` (3 branches: two `then` use h1/h2
to read the target, final `else` is `absurd rfl hne` / `rfl`). For IsValidSupp the
two `then` branches are `exfalso; obtain …; subst …; exact hnv ⟨…omega…⟩`.

## L2 — bundle the unbundled independence into `LinearIndependent`

### Index encoding (matters)
Index the family `{k // k ∈ Finset.Icc 1 (n-1)} ⊕ Fin 5`, NOT `Fin (n-1) ⊕ Fin 5`.
Payoff: the `A`-sum reindex becomes a single `Finset.sum_coe_sort` (no `+1`
offset reindex), and `coeff_A_mem`'s two bounds come free from `Finset.mem_Icc.mp p.2`.
Boundary part `![…] : Fin 5 → kerΦ`; per-index value lemmas are `rfl`:
```lean
@[simp] lemma genFamily_inr0_val (hn) : (genFamily hn (Sum.inr 0)).val = coeff_B := rfl  -- ![] index is defeq
```

### The bridge proof
```lean
rw [Fintype.linearIndependent_iff]; intro c hc
-- STEP 1: kerΦ-sum → pointwise V equation. ★ Use `.val`, NOT `(x : V F)`:
have key1 : ∀ a b u v, ∑ i, c i * ((genFamily _ i).val a b u v) = 0 := by
  intro a b u v
  have h := congrArg (fun x : kerPhi _ => x.val a b u v) hc
  simpa only [Submodule.coe_sum, SetLike.val_smul, Finset.sum_apply, Pi.smul_apply,
    smul_eq_mul, ZeroMemClass.coe_zero, Pi.zero_apply] using h
-- STEP 2-3: split + align with the unbundled lemma's hypothesis shape
set cA : ℕ → F := fun k => if h : k ∈ Finset.Icc 1 (n-1) then c (Sum.inl ⟨k,h⟩) else 0 with hcA
have hcaeq : ∀ p : {k // k ∈ Finset.Icc 1 (n-1)}, cA p.val = c (Sum.inl p) := by
  intro p; simp only [hcA]; rw [dif_pos p.2, Subtype.coe_eta]
have hsum : ∀ si sj tu tv, (∑ k ∈ Finset.Icc 1 (n-1), cA k * coeff_A k …) + … = 0 := by
  intro si sj tu tv
  have hk := key1 si sj tu tv
  rw [Fintype.sum_sum_type, Fin.sum_univ_five] at hk
  simp only [genFamily_inl_val, genFamily_inr0_val, …] at hk
  have hA : (∑ k ∈ Icc, cA k * coeff_A k …) = ∑ p : {k//Icc}, c (Sum.inl p) * coeff_A p.val … := by
    rw [← Finset.sum_coe_sort (Finset.Icc 1 (n-1)) (fun k => cA k * coeff_A k …)]
    exact Finset.sum_congr rfl (fun p _ => by rw [hcaeq p])
  rw [hA]; linear_combination hk           -- ★ NOT linarith (field, unordered); absorbs assoc/comm
obtain ⟨hAz, hB, hC, hD, hE, hF⟩ := indep_core hn cA (c (Sum.inr 0)) … hsum
intro i; cases i with
| inl p => have h0 := hAz p.val p.2; rwa [hcaeq p] at h0
| inr j => fin_cases j <;> first | exact hB | exact hC | exact hD | exact hE | exact hF
```

## L3 — finrank ≥ card

```lean
haveI : FiniteDimensional F (kerPhi hn) := (bridge_equiv hn).symm.finiteDimensional  -- infinite ambient ⇒ NOT free
have hcard := (genFamily_linearIndependent hn).fintype_card_le_finrank
simp only [Fintype.card_sum, Fintype.card_coe, Fintype.card_fin, Nat.card_Icc] at hcard
omega   -- finrank is an opaque ℕ atom; (n-1)+1-1+5 = n+4
```

## Pitfalls (each cost a real iteration)

- **`(x : V F)` does NOT insert the SetLike coe for a Submodule element.** You get
  `type mismatch: genFamily i has type ↥(kerPhi) but expected V F`. Use `x.val` /
  `(genFamily _ i).val`. Keep the whole proof in `.val`/`SetLike.val_smul` form.
- **`linear_combination hk`, not `linarith`.** Field F is unordered → linarith fails.
  `linear_combination` closes the goal-vs-`hk` mismatch from `Fintype.sum_sum_type`
  (left-assoc `(((f0+f1)+f2)+f3)+f4`) vs the goal's `t0+t1+…` (different `+` nesting);
  the big `∑ p` is treated as one ring atom (identical on both sides after `rw [hA]`).
- **Do NOT restate `Fintype.card (explicit ⊕ type)` in a fresh `have`** — it triggers a
  fresh `Fintype` instance synthesis that fails (`failed to synthesize Fintype (↥(Finset.Icc …) ⊕ Fin 5)`,
  subtype-vs-`↥s` instance diamond). Instead `simp only [Fintype.card_sum, Fintype.card_coe,
  Fintype.card_fin, Nat.card_Icc] at hcard` — rewrite the EXISTING hyp whose instance is already fixed.
- **FiniteDimensional of a submodule of an INFINITE Pi-space is not `infer_instance`.**
  Import it through the bridge: `(bridge_equiv hn).symm.finiteDimensional` (the finite
  side is `ConstraintKernel ⊆ AdjData`, finite, so f.d. is free there).
- **Duplicate proof block from a fuzzy `patch`.** A `patch` whose `old_string` ends in
  `  sorry` can fuzzy-match when the file no longer has that `sorry` (an earlier partial
  attempt lives there), inserting your block and leaving the old one dangling AFTER the
  proof completes → "no goals". ALWAYS `read_file` the full proof region before patching
  near a sorry / incremental-dev boundary; after an ambiguous patch, re-read and dedup.
- **Hard lemmas: skeleton-first.** Write `statement := by … ; sorry`, compile to confirm
  the `iff`/types apply, inspect the goal with `lean_goal`, THEN fill — do not write the
  whole fiddly body blind.
