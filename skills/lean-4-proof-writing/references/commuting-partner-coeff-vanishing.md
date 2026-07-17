# Commuting-partner canonical bracketIdentity: coefficient-vanishing recipe

For proving `coeffOf D i j u v = 0` (a 1/2-derivation coefficient vanishes) in the N_n
half-derivation project, via a chosen COMMUTING partner source. Verified 2026-06-25
(Gate 1/2 representatives `boundary_rep_A`/`transfer_rep_A`/`interior_rep_A`/
`boundary_right_rep_A`), all axiom-clean ({propext, Classical.choice, Quot.sound}).

## Recipe

1. **Commuting partner** `(c,d)`: need `j ≠ c ∧ i ≠ d`. Then
   `D.bracket_zero i j c d u v (hi hij hjn) (hc hcd hdn) (hu huv hvn) (hj_ne_c) (hi_ne_d)`
   gives `bracketIdentity D.coeff i j c d u v = 0` **directly**.
   - `HalfDerivation.bracket_zero` is in **Core** (`HalfDerivation.lean`, opened by `open
     HalfDerivation`). Do NOT `import E.Classification.Spanning` just for its U1
     `bracketSource_commuting_eq_zero` — `bracket_zero` is the Core packaging and keeps
     the file off the heavy upper-bound dependency edge.
2. `rw [bracketIdentity_eq_expanded] at h` — Core, **coeff-level** lemma taking the raw
   `(coeff)(i j c d u v)` (there is also a legacy D-level one in `HalfDerivation/Semantics.lean`;
   use the Core coeff-level one). The four terms, in order:
   `T1 = coeff i j u c`, `T2 = coeff i j d v`, `T3 = coeff c d j v`, `T4 = coeff c d u i`,
   each under a guard (`u<c∧v=d`, `u=c∧d<v`, `u=i∧j<v`, `u<i∧v=j`).
3. Resolve the four if-guards with **named** `have c1..c4`, then
   `rw [if_pos c1, if_neg c2, if_pos c3, if_neg c4] at h`. (Referee strict-mode; this
   avoids the `linter.style.whitespace` noise that inline `⟨…⟩`-in-`rw` triggers.)
   Guards involving `n`: `by omega`. Positive conjunctions: `⟨by omega, rfl⟩` or
   `⟨rfl, by omega⟩`. Negatives (false conjunction): `by omega`.
4. **Bridge `coeffOf ↔ D.coeff`** with `coeffOf_f D i j u v (6 validity hyps)`
   (`HalfProjection`): valid indices ⟹ `coeffOf D i j u v = D.coeff i j u v`. Rewrite the
   GOAL (`rw [coeffOf_f …]`) and each residual (`rw [← coeffOf_f …]`).
5. **Kill the residual**, by class:
   - wide `∈ I` residual → FROZEN `WidthFiltration.width_stability_a` (target `(1,n-1)`) /
     `width_stability_c` (target `(2,n)`). These REQUIRE `hI : I_filtered D.coeff hn`.
   - smaller-target-width `∉I` residual → the main induction `ih`.
   - target `u > i` with a good partner → **zero residual** (`T4` dead since `u<i` false;
     a partner makes `T2,T3` vanish, leaving only the target term). This variant is even
     `I_filtered`-free (no `hI` needed) — flag the unused `hI` as `_hI`.
6. **Close with `linear_combination h - hres`** (or `linear_combination hb`).
   **NOT `linarith`** — `F` is an arbitrary `[Field F]` with NO order; `linarith` fails.
   `linear_combination` works over any commutative ring.

## Pitfalls

- `linarith` on a general `[Field F]` goal **FAILS** (no order). Always `linear_combination`.
- `n-1` indices are omega-friendly; keep the named-have guard resolution and `by omega`.
- A boundary residual landing on `width_stability` carries `hI : I_filtered`. If the HOST
  theorem must be hI-free (because it ESTABLISHES I-containment, e.g. `imageContainment` /
  `centered_image_in_I`), this is a real STATEMENT-LEVEL conflict, not a missing step —
  and can be characteristic-dependent (det-of-coupling). See
  `lean-4-workflow / references/derisk-final-assembly-gates.md` (Gate 3/4/5 + char audit).
- `coeffOf_f`'s long `(by omega)` × 6 arg line can exceed the 100-col linter; wrap the arg
  list across two lines (a continuation of `rw [` does NOT trip the whitespace linter).
- The `unusedSectionVars [CharNeTwo F]` warning on these lemmas is a known false-friend
  (the instance is needed for synthesis through `bracketIdentity_eq_expanded`); leave it.
