# finrank UPPER bound from a spanning set (DecidableEq-free)

Complements the lower-bound/independence references (`finrank-from-coeff-independence.md`,
`finrank-submodule-assembly.md`). Use when proving `finrank (S : Submodule F M) ≤ N`
from a finite spanning set — **especially when the ambient `M` is a function/Pi type**
(`ℕ→ℕ→ℕ→ℕ→F`, `Fin k → F`, …) that has **no `DecidableEq`**.

## The assembly chain

Given `hspan : S ≤ Submodule.span F G` (a spanning lemma) and a finite `G : Set M`:

```lean
-- P4b: finrank (span G) ≤ N
haveI : Fintype ↥G := G_finite.fintype                    -- Set.Finite.fintype (uses Classical)
let g : Fin a ⊕ Fin b → ↥G := fun x => match x with      -- explicit surjection onto ↥G
  | Sum.inl i => ⟨gen_i i, mem_proof_i⟩
  | Sum.inr j => ⟨![g0, g1, …] j, by fin_cases j <;> simp [G]⟩
have hsurj : Function.Surjective g := by
  rintro ⟨v, hv⟩
  rcases hv with hvr | hvbf            -- union: v ∈ range(..) ∨ v ∈ {literals}
  · obtain ⟨i, rfl⟩ := hvr; exact ⟨Sum.inl i, rfl⟩
  · simp only [Set.mem_insert_iff, Set.mem_singleton_iff] at hvbf
    rcases hvbf with rfl|rfl|… ; exacts [⟨Sum.inr 0, rfl⟩, ⟨Sum.inr 1, rfl⟩, …]
calc Module.finrank F (Submodule.span F G)
    ≤ G.toFinset.card        := finrank_span_le_card _
  _ = Fintype.card ↥G        := Set.toFinset_card _
  _ ≤ Fintype.card (Fin a ⊕ Fin b) := Fintype.card_le_of_surjective g hsurj
  _ = a + b                  := by rw [Fintype.card_sum, Fintype.card_fin, Fintype.card_fin]
  _ = N                      := by omega

-- P4c: finrank S ≤ N
haveI : Module.Finite F ↥(Submodule.span F G) := Module.Finite.span_of_finite F G_finite
exact le_trans (Submodule.finrank_mono hspan) finrank_span_G_le   -- finrank_mono needs the Module.Finite above

-- P4d: finrank S = N
exact le_antisymm finrank_S_le finrank_S_ge                       -- ge = the (separate) lower bound
```

## The DecidableEq trap (the whole reason for the surjection detour)

`Set.toFinset_union`, `Set.toFinset_range`, `Finset.card_union_le` **all require
`[DecidableEq M]`** — a function-type `M` has none, so you canNOT compute
`G.toFinset.card` by splitting the union/range. Route around it:
`finrank_span_le_card` → `Set.toFinset_card` (`= Fintype.card ↥G`, **no DecidableEq**)
→ `Fintype.card_le_of_surjective` from an explicit `Fin a ⊕ Fin b ↠ ↥G`.

`G` finiteness: `(Set.finite_range _).union (Set.toFinite _)`.
`Set.finite_insert` is an **iff, not a function** — use `Set.toFinite _` for finite
literal sets `{a,b,c,…}`.

For `Module.Finite` of the submodule `S` itself when the ambient is infinite-dim
(e.g. a coeff Pi-space): transport it, `(bridge_equiv).symm.finiteDimensional`
(see `representation-bridge-finrank-carrier.md`). For `span G` it is free:
`Module.Finite.span_of_finite F G_finite`.

## API recon — mathlib v4.31.0-rc1 (verified live / dead, 2026-06)

LIVE:
- `finrank_span_le_card (s : Set M) [Fintype ↑s] : finrank R (span R s) ≤ s.toFinset.card`
- `Submodule.finrank_mono [Module.Finite R ↥t] : s ≤ t → finrank s ≤ finrank t`
- `Module.Finite.span_of_finite (R) {A : Set M} : A.Finite → Module.Finite R ↥(span R A)`
- `FiniteDimensional.span_of_finite (K) {A} : A.Finite → FiniteDimensional K ↥(span K A)`
- `Set.toFinset_card (s) [Fintype ↑s] : s.toFinset.card = Fintype.card ↑s`  (no DecidableEq)
- `Set.Finite.fintype : s.Finite → Fintype ↑s`
- `Fintype.card_le_of_surjective (f) : Surjective f → card β ≤ card α`
- `Fintype.card_sum`, `Fintype.card_fin`, `Set.ncard_union_le`, `Nat.card_eq_fintype_card`

DEAD (do not reach for these — unknown constants):
- `Set.card_range_le`, `Set.Nat.card_coe_set_eq`, `Set.Finite.finrank_span_le`,
  `Submodule.finiteDimensional_span_of_finite`
- `Set.finite_insert` exists but is an **iff** — not applicable as a function.

## Notes

- Prefer a **V F-level** generator `Set` `{gen_i} ∪ {literals}` over a bundled
  `Sum`/`Fin`-indexed `genFamily` when matching paper-level `span{…}` — it avoids a
  Sum/Fin index-encoding conversion layer. (The lower bound may still use a bundled
  family; the two need not match.)
- The upper bound can be computed **directly on the target Submodule** via
  `finrank_mono`, bypassing any `≃ₗ`/range bridge used elsewhere for the lower bound.
