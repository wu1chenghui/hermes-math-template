# finrank of a coeff-Pi-space Submodule — the final-assembly layer

Proving `finrank (kerΦ) = n + k` where `kerΦ : Submodule F (V F)`,
`V F = ℕ→ℕ→ℕ→ℕ→F` (coeff Pi-space), and you already have a representation
bridge `bridge_equiv : kerΦ ≃ₗ[F] ConstraintKernel` to a FINITE coordinate model
(`ConstraintKernel ⊆ AdjData`, `AdjData` a finite product of `F`). This is the
`DimensionTheorem.lean` layer; the techniques transfer to any "dimension of a
constraint solution space" assembly. Pairs with
`references/representation-bridge-finrank-carrier.md` (which builds the bridge);
this file computes the finrank ON that carrier.

## 0. Machine-probe the scaffolding BEFORE writing proofs
Add a temporary probe block in the importing file, compile, read diagnostics /
`lean_goal`, then REVERT to a clean skeleton (don't leave probes in canon files):
```lean
example : FiniteDimensional F (AdjData F n) := by infer_instance          -- finite product → FREE
example : Module.finrank F (AdjData F n) = (n-1)+((n-1)+(n-1)) := by
  simp [AdjData, Module.finrank_prod]                                      -- finite-model dim, FREE
example : FiniteDimensional F (ConstraintKernel (F := F) hn) := by infer_instance  -- submodule of finite → FREE
example : FiniteDimensional F (kerPhi (F := F) hn) := by infer_instance    -- FAILS: kerΦ ⊆ infinite V
example : FiniteDimensional F (kerPhi (F := F) hn) :=
  (bridge_equiv (F := F) hn).symm.finiteDimensional                        -- transport, 1 line
example : Module.finrank F (kerPhi (F:=F) hn) = Module.finrank F (ConstraintKernel (F:=F) hn) :=
  (bridge_equiv (F:=F) hn).finrank_eq                                      -- D2 = D1, unconditional 1-liner
```

### ★ F-pinning gotcha
`infer_instance` / a proof term over a def that carries `[CharNeTwo F]` (or any
field-class) in its signature fails with
`typeclass instance problem is stuck — CharNeTwo ?m … arguments are metavariables`
EVEN THOUGH `[CharNeTwo F]` is in scope: `F` wasn't pinned before instance search
ran. FIX: annotate — `ConstraintKernel (F := F) hn`, `bridge_equiv (F := F) hn`,
`kerPhi (F := F) hn`. Plain `#check`s elaborate fine (no instance search); only
`infer_instance`/proof terms need the annotation.

### ★ finite-dimensionality is NOT uniform
- Submodule of a FINITE-dim ambient (finite product `Fin k → F`, or `×` of them)
  ⇒ f.d. is FREE (`infer_instance`).
- Submodule of an INFINITE Pi-space (`ℕ⁴→F`) ⇒ NOT auto f.d.; `infer_instance`
  genuinely fails. Import it via the bridge: `(equiv).symm.finiteDimensional`.
- `LinearEquiv.finrank_eq` is unconditional (no f.d. hyp) — the D2-from-D1
  transfer `finrank kerΦ = finrank ConstraintKernel` is one line.

## 1. Generator membership in the Submodule (lower-bound step "GAP-L1")
Predicate `IsKerPhi := I_filtered c ∧ IsValidSupp c ∧ (Φ c = 0, validity-guarded)`.
Reusable core (discharges Φ=0 from an existing `HalfDerivation` witness):
```lean
lemma mem_kerPhi_of (hn : 3 ≤ n) (D : HalfDerivation F n)
    (hI : I_filtered D.coeff hn) (hV : IsValidSupp (n := n) D.coeff) :
    D.coeff ∈ kerPhi (F := F) hn := by
  refine ⟨hI, hV, ?_⟩
  intro i j p q u v h1 h2 h3 h4 h5 h6 h7 h8 h9
  exact halfDerivation_phi_zero D i j p q u v h1 h2 h3 h4 h5 h6 h7 h8 h9
```
- `refine ⟨_,_,_⟩` works: Submodule membership reduces defeq to the predicate `And`.
- `(halfDeriv_X i).coeff` is defeq `coeff_X i`, so
  `mem_kerPhi_of hn (halfDeriv_X …) (coeff_X_I_filtered …) (coeff_X_validSupp …)`
  proves `coeff_X … ∈ kerPhi` directly (no rewrite needed).

`I_filtered` (`coeff i j u v ≠ 0 → (u,v) ∈ I_target_set`):
```lean
intro a b u v hne
simp only [coeff_X] at hne
split_ifs at hne with h          -- single-if: `with h`; double-if: `with h1 h2`
· obtain ⟨_, _, hu, hv⟩ := h; subst hu; subst hv; simp [I_target_set]
· exact absurd rfl hne            -- final else branch: hne : (0:F) ≠ 0
```
`IsValidSupp` (`¬valid (a,b,u,v) → coeff a b u v = 0`), SINGLE-if:
```lean
intro a b u v hnv
simp only [coeff_X]
rw [if_neg]
rintro ⟨ha, hb, hu, hv⟩
subst ha; subst hb; subst hu; subst hv
exact hnv ⟨by omega, by omega, by omega, by omega, by omega, by omega⟩
```
DOUBLE-if (`split_ifs` instead of `rw [if_neg]`; both `then` branches `exfalso`):
```lean
intro a b u v hnv
simp only [coeff_X]
split_ifs with h1 h2
· exfalso; obtain ⟨ha,hb,hu,hv⟩ := h1; subst ha; subst hb; subst hu; subst hv
  exact hnv ⟨by omega, by omega, by omega, by omega, by omega, by omega⟩
· exfalso; obtain ⟨ha,hb,hu,hv⟩ := h2; subst ha; subst hb; subst hu; subst hv
  exact hnv ⟨by omega, by omega, by omega, by omega, by omega, by omega⟩
· rfl
```
Validate each with `lean_diagnostic_messages(severity=error)`: prove the single-if
template once (one generator), then one double-if (confirms the new branch shape),
then batch the rest — they are byte-for-byte the same modulo the `coeff_X` name.

## 2. Bundle the unbundled `indep_core` into `LinearIndependent` ("GAP-L2")
`indep_core` is stated coordinate-wise (`∀ idx, Σ cA k·coeff_A k + cB·coeff_B+… = 0
→ all scalars 0`), NOT as `LinearIndependent`. Bridge:

### ★ Index encoding: `{k // k ∈ Finset.Icc 1 (n-1)} ⊕ Fin 5`, NOT `Fin (n-1) ⊕ Fin 5`
The `Icc`-subtype index makes the A-sum reindex a single `Finset.sum_coe_sort`
(matches `indep_core`'s `∑ k ∈ Icc 1 (n-1)` with NO `+1` offset) AND supplies
`coeff_A_mem`'s two bounds directly via `(Finset.mem_Icc.mp p.2).1/.2` (no `omega`).
`Fintype.card ({k // k∈Icc 1(n-1)} ⊕ Fin 5) = (n-1)+5 = n+4`
(`Fintype.card_sum`, `Fintype.card_coe`, `Finset.card_Icc`).
```lean
noncomputable def genFamily (hn : 3 ≤ n) :
    {k // k ∈ Finset.Icc 1 (n-1)} ⊕ Fin 5 → kerPhi (F := F) hn :=
  Sum.elim
    (fun p => ⟨_, coeff_A_mem hn p.val (Finset.mem_Icc.mp p.2).1 (Finset.mem_Icc.mp p.2).2⟩)
    (fun j => (![ (⟨_, coeff_B_mem hn⟩ : kerPhi (F:=F) hn),
                  ⟨_, coeff_C_mem hn⟩, ⟨_, coeff_D_mem hn⟩,
                  ⟨_, coeff_E_mem hn⟩, ⟨_, coeff_F_mem hn⟩ ]) j)
```
Proof spine (`Fintype.linearIndependent_iff`):
1. `rw [Fintype.linearIndependent_iff]; intro c hc` → `hc : ∑ i, c i • genFamily i = 0`
   (in the Submodule), goal `∀ i, c i = 0`.
2. Drop to V pointwise: `Subtype.ext_iff` + `Submodule.coe_sum` + `coe_smul`,
   then `congrFun ×4` + `Finset.sum_apply` + `Pi.smul_apply` (`smul = mul` on `F`-fns).
3. `Fintype.sum_sum_type` splits A-part (subtype sum) + Fin-5 boundary
   (`Fin.sum_univ_five`); `Finset.sum_coe_sort` turns the A-part into `∑ k ∈ Icc 1(n-1)`.
4. Set `cA k := if h : k ∈ Finset.Icc 1 (n-1) then c (Sum.inl ⟨k,h⟩) else 0`,
   match `indep_core`'s hypothesis, `apply indep_core` → all scalars 0.
5. Recover `∀ i`: `inl` from the `∀ k ∈ Icc` conjunct, `inr` from the 5 conjuncts.

Lock the statement first with `:= by … ; sorry`, confirm the iff/setup elaborates,
inspect the post-`intro` goal with `lean_goal`, THEN fill — never write the reindex blind.

## 3. finrank lower bound ("GAP-L3", mechanical)
`(bridge_equiv hn).symm.finiteDimensional` supplies `[FiniteDimensional F (kerPhi hn)]`;
then `LinearIndependent.fintype_card_le_finrank genFamily_linearIndependent`
⇒ `n+4 = Fintype.card _ ≤ finrank F (kerPhi hn)`. The upper bound (`≤ n+4`) is the
separate spanning lemma (paper design in BASIS-SPANNING); `le_antisymm` closes it.

## Pitfalls
- `I_filtered`/`IsValidSupp` lemmas trip the `unused [CharNeTwo F]` section-var linter
  (pure index facts, no field arithmetic). Cosmetic; `omit [CharNeTwo F] in` if you care.
- Thread the `n ≥ 3` vs `n ≥ 4` boundary: `kerPhi`/membership need `3 ≤ n`,
  `indep_core` needs `4 ≤ n`. State the bundled lemma with `4 ≤ n` and pass
  `(show 3 ≤ n by omega)` to `genFamily`; Prop proof-irrelevance makes the two
  `hn` proof terms interchangeable.
