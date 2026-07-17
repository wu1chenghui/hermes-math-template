# Sparse-coefficient bracket identities & support-disjointness independence

Reusable tactics for proving facts about **sparse `if`-guarded coefficient
functions** `coeff : ℕ → ℕ → ℕ → ℕ → F` (the half-derivation / Lie-bracket
classification project, but the patterns generalize to any goal that is a
linear combination of deeply-nested `if (a ∧ b ∧ c ∧ d) then v else 0` terms).

Proven on N_n half-derivation cocycles (2026-06-22, Lean 4.31, mathlib). Every
technique below was compiled to 0 error / 0 sorry / 0 warning.

---

## 1. The `split_ifs` explosion — root cause and the fix

A coefficient like
```lean
def coeff_E (si sj tu tv : ℕ) : F :=
  if si = n-1 ∧ sj = n ∧ tu = 1 ∧ tv = n-1 then 1 else 0
```
plugged into a bracket identity (`bracketSource`/`bracketLeft`/`bracketRight`)
gives a goal with ~6 outer `if`-guards, each wrapping a 4-conjunction `if`.

**Trap:** `simp [ha, hb, ...]` (general simp, or simp with hypotheses) UNFOLDS
each 4-conjunction guard into 4 *independent* `if`s. Then `split_ifs` faces
~15–18 independent boolean conditions → **2^18 ≈ 262144 subgoals → deterministic
timeout at `whnf`**, even at `maxHeartbeats 1000000`.

**Fix:** use `simp only [bracketSource, bracketIdentity, bracketLeft,
bracketRight, coeff_X]`. `simp only` with just the defs keeps each 4-conjunction
as ONE `Decidable` proposition, so `split_ifs` sees ~12 conditions
(2^12 ≈ 4096) — the same tractable order as a single-`if` coefficient.

**Winning template (single-`if` coefficients):**
```lean
simp only [bracketSource, bracketIdentity, bracketLeft, bracketRight, coeff_X]
split_ifs <;> first | rfl | (exfalso; omega) | ring
```
**Closer order matters:** put `(exfalso; omega)` BEFORE `ring`. The vast majority
of branches are unreachable index-contradictions that `omega` closes; if `ring`
runs first it leaks "Try this: ring_nf" noise on every contradiction branch.

---

## 2. Double-`if` coefficients: kill the zero terms, then reduce dimension

Coefficients with two active sources (two nested `if`s, e.g. `coeff_C/D/F`)
still hit 2^18 with the bare template. Strategy:

1. Identify the **two bracket sub-terms that are identically zero** (this differs
   per coefficient and encodes the algebra — see below).
2. Kill each with a LOCAL `have` (each is a 3-condition split → 8 branches,
   instant): `have hX : (<exact if-expr>) = 0 := by split_ifs <;> first | rfl | (exfalso; omega)`
3. `rw [hX, hY]` them away → the residual `split_ifs` only sees the 4 surviving
   terms (2^12 ≈ 4096, tractable).

Which two terms vanish reflects the structure (observed on N_n cocycles):
- `c=n`/`a=n` inner condition contradicts `c<d≤n`/`a<b≤n` (column-n targets).
- `d=1`/`b=1` inner condition contradicts `c<d`/`a<b` (row-1 targets).
- For a "commuting-source" cocycle the WHOLE `bracketSource ≡ 0` (both source
  terms vanish) — e.g. `[E_12, E_{n-1,n}] = 0`.

For coefficients with `½` (needs `2⁻¹`), `2`, `-1` factors: add
`have h2 : (2:F) ≠ 0 := CharNeTwo.char_ne_two` and finish residual branches with
`first | rfl | (exfalso; omega) | (field_simp; ring)`.

---

## 3. Compile-probe to get the EXACT post-`simp only` goal text

To write `have`/`rw` whose LHS matches the goal verbatim (rw is whitespace- and
form-sensitive), do NOT guess the simp output:

1. Temporarily set the proof body to JUST `simp only [..., coeff_X]` (no closer,
   no `sorry`).
2. Compile — the "unsolved goals" error prints the exact normalized goal.
3. Copy the two zero-term `if` expressions verbatim into the `have` statements.

This guarantees `rw` matches and is faster than iterating on syntax mismatches.
`lake env lean <file>` gives a ~70s single-file feedback loop (vs full `lake build`).

---

## 4. Support-disjointness: linear independence of sparse cocycles

To prove `{v_1,…,v_k}` (as coefficient functions, which live in the Pi-type
`ℕ→ℕ→ℕ→ℕ→F`, automatically a `Module F`) are linearly independent, the
lightweight form avoids `LinearIndependent`/`Finset` machinery:

```lean
theorem indep (hn : 4 ≤ n) (c1 … ck : F)
    (h : ∀ si sj tu tv, c1 * coeff_1 .. + … + ck * coeff_k .. = 0) :
    c1 = 0 ∧ … ∧ ck = 0 := by
  have h2 : (2:F) ≠ 0 := CharNeTwo.char_ne_two
  refine ⟨?_, …, ?_⟩
  -- one bullet per scalar, specialised at that cocycle's UNIQUE identifying point
  · have e := h <identifying point of v_i>
    simp [coeff_1, coeff_2, …, coeff_k] at e
    split_ifs at e <;> first | (exfalso; omega) | ((try field_simp at e); simpa using e)
```

**Engine:** each cocycle has a unique `(source, target)` index where it is the
ONLY nonzero one. Specialise `h` there; all other terms are 0 (or a residual
`if` that `omega` + `hn` kills); the surviving `c_i * value = 0` solves to `c_i = 0`.

Key tactic choices:
- Use **full `simp [coeff_*] at e`** here (NOT `simp only`): at a CONCRETE point
  it evaluates every "pure-numeral" conjunction (`2=3 → False → 0`,
  `1=1∧2=2∧… → True → value`), leaving only `if`s with the symbolic `n`
  (e.g. `1 = n-2`). Those residual `if`s have LINEAR negations, so
  `split_ifs at e <;> (exfalso; omega)` closes their unreachable branches using `hn`.
- `((try field_simp at e); simpa using e)` for the if-false branch: `try`
  swallows `field_simp`'s "made no progress" on `±1`-coefficient terms (no
  fraction) so it doesn't error; `field_simp` clears `2⁻¹`/`2` on fractional
  terms; `simpa` finishes. If a particular branch is purely `c = 0` (no
  fraction), use `simpa using e` alone to avoid a `field_simp does nothing` lint
 warning.

 ### 4b. Family-indexed generators (a `Finset` sum inside the combination)

 When the generating set includes an **n-indexed family** (e.g. center maps
 `A_k`, k = 1..n-1) alongside the fixed boundary cocycles, the linear
 combination carries a `Finset` sum and the §4 "avoid Finset" form no longer
 applies. Handle the sum with two collapse lemmas:

 ```lean
 theorem indep_core (hn : 4 ≤ n) (cA : ℕ → F) (cB … cF : F)
   (h : ∀ si sj tu tv,
     (∑ k ∈ Finset.Icc 1 (n-1), cA k * coeff_A k si sj tu tv)
     + cB * coeff_B .. + … = 0) :
   (∀ k ∈ Finset.Icc 1 (n-1), cA k = 0) ∧ cB = 0 ∧ … := by
 ```

 - **At the family's OWN identifying point** (separate `cA p`): specialise `h`
 at `(p, p+1, 1, n)`, then collapse the sum to the single surviving term with
 `Finset.sum_eq_single_of_mem`, supplying the "others vanish" proof inline:
 ```lean
 have hAsum : (∑ k ∈ Finset.Icc 1 (n-1), cA k * coeff_A k p (p+1) 1 n) = cA p := by
   rw [Finset.sum_eq_single_of_mem p (Finset.mem_Icc.mpr hp)
     (fun k _ hkp => by
       simp only [coeff_A]
       rw [if_neg (by rintro ⟨hpk, _, _, _⟩; exact hkp hpk.symm)]; ring)]
   simp [coeff_A]   -- main term: coeff_A p p (p+1) 1 n = 1, so cA p * 1 = cA p
 rw [hAsum] at e
 ```
 - **At ANOTHER cocycle's identifying point** (separating `cB…cF`): the whole
 family sum vanishes there, because that point's `tu`/`tv`/source matches NO
 family member. Kill it with `Finset.sum_eq_zero` and rewrite, then proceed
 exactly as the §4 boundary closer:
 ```lean
 have hAz : (∑ k ∈ Finset.Icc 1 (n-1), cA k * coeff_A k 1 2 2 n) = 0 := by
   apply Finset.sum_eq_zero; intro k _
   simp only [coeff_A]; rw [if_neg (by rintro ⟨_, _, h3, _⟩; omega)]; ring
 rw [hAz] at e
 ```
 The contradictory clause differs per point (`tu=2≠1`, `tv=n-1≠n`, or two
 source clauses like `1=k ∧ 3=k+1`) — a uniform `rintro ⟨h1,h2,h3,h4⟩; omega`
 (with `hn` in context) finds whichever it is.

 This whole proof compiled 0 sorry / 0 warning on the first try once the two
 collapse lemmas were in place; `lean_verify` confirmed `propext / Classical.choice /
 Quot.sound` only. The family form is the machine realization of the
 "evaluation morphism → coordinate injectivity" lower bound (see paper-engineering
 `references/exact-dimension-closure.md`).

 ---

 ## 5. Pitfalls discovered

- **`exfalso; omega` cannot close `¬(1=1 ∧ 2=2 ∧ n=n)`.** When `split_ifs`
  generates the else-branch of a TRIVIALLY-TRUE conjunction guard, its
  hypothesis is a negated conjunction containing reflexive equalities like
  `n=n`; omega does not refute it. **Fix:** use full `simp` (not `simp only`)
  BEFORE `split_ifs` so trivially-true conjunctions are pre-evaluated to `True`
  and never reach `split_ifs`. (This is why §4 uses full `simp` while §1–2 use
  `simp only`: §1–2 work on SYMBOLIC indices where no conjunction is decidable;
  §4 works at CONCRETE points where most are.)
- **`field_simp` errors with "made no progress"** on a goal without fractions.
  Wrap as `(try field_simp at e)` or branch so fraction-free coefficients use
  `simpa` directly.
- **`ring` does not know `2 * 2⁻¹ = 1`** (it does not assume `2 ≠ 0`). For
  `Field F` with `CharNeTwo`, supply `have h2 : (2:F) ≠ 0 := CharNeTwo.char_ne_two`
  and use `field_simp` (it picks up `h2` from context) before `ring`.
- **Lint warnings `'tactic does nothing' / 'never executed'`** mean a `first |
  …` alternative is dead code in that branch. To reach 0 warnings you often must
  split one bullet's closer from the others (e.g. the `±1`-coefficient bullet
  drops `field_simp`, the `½`-coefficient bullets keep it). Locate a single
  bullet for a targeted edit via its unique `have e := h <point>` line.

---

## 6. MCP LSP (lean-lsp-mcp) warmup / fallback

`lean_goal`, `lean_multi_attempt`, `lean_diagnostic_messages` time out (120s)
on the FIRST call against a long-dependency file while the language server cold-
starts loading `.olean`s; they respond sub-second on small files and warm up
after a `lake build`/`lake env lean` has run. For long-dependency files they can
still time out — fall back to the **compile-probe** (§3) via
`lake env lean <file>` for goal inspection. `lean_verify <Namespace.thm>` is
worth a final axiom check (confirms `propext / Classical.choice / Quot.sound`
only, i.e. no `sorryAx`).
