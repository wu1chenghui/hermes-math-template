# finrank, cardinality bookkeeping & layer-interface Lean nuggets

Concrete, reusable Lean 4 (mathlib v4.31.0-rc1) techniques from a
finrank-of-a-coeff-Submodule proof. The recurring lesson: in finrank/dimension
"assembly" the work is ~90% finding the right mathlib name + instance, ~10% math.

## API recon FIRST (`#check @name` to split live from dead)
Before writing an assembly step, `#check @candidate` in `lean_run_code`. Confirmed
this session:
- LIVE `finrank_span_le_card (s : Set M) [Fintype ↑s] : finrank (span s) ≤ s.toFinset.card`
- LIVE `Submodule.finrank_mono {s t} [Module.Finite ↥t] : s ≤ t → finrank s ≤ finrank t`
- LIVE `Module.Finite.span_of_finite (R) {A} : A.Finite → Module.Finite ↥(span R A)`
- LIVE `Set.toFinset_card (s) [Fintype ↑s] : s.toFinset.card = Fintype.card ↑s`
- LIVE `Fintype.card_le_of_surjective`, `Fintype.card_sum`, `Fintype.card_fin`,
  `Set.Finite.fintype`, `Set.ncard_union_le`, `Nat.card_eq_fintype_card`
- DEAD (do NOT guess): `Set.card_range_le`, `Set.Nat.card_coe_set_eq`,
  `Set.Finite.finrank_span_le`, `Submodule.finiteDimensional_span_of_finite`.
- `Set.finite_insert` is an **iff**, NOT a function — build `Finite` of
  `range f ∪ {a,b,…}` via `(Set.finite_range _).union (Set.toFinite _)`.

## DecidableEq-free cardinality upper bound (function-valued ambient)
When the ambient is a function type (`V F = ℕ→ℕ→ℕ→ℕ→F`) there is NO `DecidableEq`,
so `Set.toFinset_union`, `Set.toFinset_range`, `Finset.card_union_le` (all require
`[DecidableEq α]`) are unavailable. Route around it:
```
finrank (span s) ≤ s.toFinset.card     -- finrank_span_le_card  (needs [Fintype ↑s] := s_finite.fintype)
                 = Fintype.card ↥s       -- Set.toFinset_card     (NO DecidableEq)
                 ≤ Fintype.card ι        -- Fintype.card_le_of_surjective g hg, explicit surjection g : ι ↠ ↥s
                 = known number          -- Fintype.card_sum + Fintype.card_fin
```
Build the surjection `g : Fin a ⊕ Fin b → ↥s` by hand (`inl i ↦ ⟨elt i, mem⟩`,
`inr j ↦ ⟨![..] j, mem⟩`); surjectivity by `rintro ⟨v,hv⟩; rcases hv … ; exact
⟨inl i, rfl⟩ / ⟨inr 0, rfl⟩ …`. Zero DecidableEq. `Module.Finite ↥(span s)` for
`finrank_mono` is free from `Module.Finite.span_of_finite F s_finite`.

## defeq / metavar / Nat-subtraction nuggets
- `set x := e with hx` makes `x` a let-binding **defeq** to `e` — `show`/`rfl` see
  through it. But a term produced *later* is NOT auto-folded to `x`; bridge with
  the `hx` equation (`rw [hx]`). Don't assume `set` is opaque (it isn't), and don't
  assume later terms fold (they don't).
- A value whose type doesn't determine an implicit ⇒ that implicit is a **metavar**.
  E.g. `evAdjFun (T : V F)` (V F doesn't mention `n`) ⇒ `n` free ⇒ a Fin literal
  `⟨n-3, by omega⟩` fails (`omega could not prove … ?m`). FIX: ascribe to pin it:
  `(evAdjFun (T : V F) : AdjData F n)`. Ascribe BEFORE constructing Fin/omega goals.
- Nat subtraction is NOT defeq: `(n-2)+1` ≠ `n-1` definitionally. `rfl`/`congr`
  fail; use `rw [show (n-2)+1 = n-1 from by omega, show (n-2)+2 = n from by omega]`.
  (Literals like `0+1=1` DO `rfl`.)
- `Function.Injective f` applies directly: from `hS : f a = f b`,
  `f_injective hS : a = b` — no manual `Subtype.ext` even when `a b` are subtype
  elements.
- Dirac-delta finite sum `∑ x, if ↑p = ↑x then bs x else 0 = bs p`:
  `simp only [Fin.val_inj, Finset.sum_ite_eq, Finset.mem_univ, if_true]`.
  Do NOT use `Finset.sum_ite_eq'` (wrong association — leaves the goal; its simp
  args report as "unused"). `Fin.val_inj` collapses `↑p=↑x` → `p=x` first.
- A goal `nf = nf` left open after a `rw` chain that differs only in `Fin` proof
  terms: it's proof-irrelevance-defeq but rw's syntactic auto-`rfl` may miss it.
  Make the params let-bound coords so both sides are literally identical, or add an
  explicit closer.

## Submodule membership of a hand-written linear combination
To show a pointwise-defined `S_val : V F` lies in a Submodule / `span G`: first
`funext` + `simp only [Pi.add_apply, Pi.smul_apply, Finset.sum_apply, smul_eq_mul]`
to rewrite it as the genuine combination `∑ c i • gen i + …`, then
`refine add_mem (add_mem … ) …` with `Submodule.sum_mem` / `smul_mem` /
`subset_span`. The SAME skeleton proves `∈ kerΦ` (via `coeff_X_mem`) and `∈ span G`
(via `subset_span (Or.inl ⟨i, rfl⟩)`).

## Engineering preference: explicit case list > parametrized generic lemma
For a family of structurally-similar lemmas (e.g. one commuting-partner per target
class), prefer an **explicit finite case list** (one thin lemma each) over a single
parametrized generic lemma — at least until the family closes. In Lean a premature
`∀ target, ∃ partner, …` degenerates into ~80% Fin/omega/witness/existence plumbing
and ~20% actual proof. Abstract into a generic lemma ONLY after the cases close and
prove to share one pattern. (User-ratified.)
