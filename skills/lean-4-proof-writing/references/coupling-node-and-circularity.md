# Same-Level Coupling Nodes & the "Don't Assume What You're Proving" Circularity

Two linked proof-architecture lessons for inductive coefficient-classification proofs
(e.g. `imageContainment`: "off-diagonal target ∉ exception-set ⟹ coeff = 0", by strong
induction on target-width). Worked example: the `ImageContainment.boundary_coupling`
lemma and the Gate-3/4/5 audit that motivated it.

## 1. Most induction nodes DESCEND; a few are SAME-LEVEL coupling nodes

In a strong induction on a measure (target-width `v−u`), the typical node reduces to
children of STRICTLY SMALLER measure → killed by the `ih`. But occasionally a node's
constraint involves another coefficient of the SAME or LARGER fixed measure that loops
back (a 2-cycle, not a descent). The `ih` cannot reach it. **Do NOT try to force it into
the induction** — recognise it as a *coupling node* and close it with an explicit small
linear system.

How to find them BEFORE writing the induction (cheap, compile-backed):
- Audit where each sub-lemma's hypotheses get discharged and at what measure (see the
  "hI use-point audit" pattern — for every use of an assumption, what does it kill and
  what's that object's measure? If all strictly-smaller ⇒ ih-replaceable; if same/larger
  ⇒ coupling node).
- Take the residual coefficient and expand its defining identity via `lean_run_code`
  probes (commuting-partner `bracket_zero` + `bracketIdentity_eq_expanded`). Read which
  terms survive; if the residual lands back on a same-measure partner, it's a 2-cycle.

## 2. Resolving a coupling node: explicit k×k system, det ≠ 0

A 2-cycle gives a small linear system in the coupled coefficients, e.g.
`x − y = 0` (from a commuting-partner `bracketIdentity = 0`) and
`x + 2y = 0` (from a `coeffOf_cond_of` split). `det [[1,−1],[1,2]] = 3`. When the working
characteristic makes `det ≠ 0`, BOTH coefficients are pinned to 0 — solved jointly, NOT
recursively. **If `det = p`, ADD a `char ≠ p` hypothesis** (`(hp : (p : F) ≠ 0)`); the
node is degenerate exactly in char p (where genuine extra solutions live — see
lean-4-workflow `references/characteristic-exception-nullspace.md`).

Lean spine (the `boundary_coupling` shape — fully hI-free):
```lean
lemma coupling (D : HalfDerivation F n) (hn5 : 5 ≤ n) (h3 : (3 : F) ≠ 0) :
    coeffOf D 1 2 3 n = 0 ∧ coeffOf D 1 3 2 n = 0 := by
  -- R_a : x = y  (commuting partner ⇒ bracketIdentity = 0)
  have hbi : bracketIdentity D.coeff 1 2 1 3 1 n = 0 :=
    D.bracket_zero 1 2 1 3 1 n (by omega) … (by omega)        -- 6 idx + 9 valid + 2 ≠
  rw [bracketIdentity_eq_expanded] at hbi
  have c1 : ¬((1:ℕ) < 1 ∧ n = 3) := by omega                 -- NAMED if-conditions,
  have c2 : (1:ℕ) = 1 ∧ 3 < n := ⟨rfl, by omega⟩             -- then rw if_pos/if_neg
  have c3 : (1:ℕ) = 1 ∧ 2 < n := ⟨rfl, by omega⟩             -- (NOT split_ifs — avoids
  have c4 : ¬((1:ℕ) < 1 ∧ n = 2) := by omega                 --  blowup + style linter)
  rw [if_neg c1, if_pos c2, if_pos c3, if_neg c4] at hbi
  have hRa : coeffOf D 1 2 3 n = coeffOf D 1 3 2 n := by
    rw [coeffOf_f D 1 2 3 n (by omega) …, coeffOf_f D 1 3 2 n (by omega) …]  -- coeffOf↔coeff bridge
    linear_combination -hbi                                   -- NOT linarith (F unordered)
  -- R_b : 2y = -x  (coeffOf_cond_of split k=2, hI-free)
  have hcond := coeffOf_cond_of D 1 3 2 n … 2 (by omega) (by omega)
  …                                                            -- same named-have + rw if
  have hRb : 2 * coeffOf D 1 3 2 n = - coeffOf D 1 2 3 n := by linear_combination hcond
  -- det = 3 ≠ 0 : 3y = 0 ⇒ y = 0 ⇒ x = 0
  have hy : coeffOf D 1 3 2 n = 0 := by
    have h3y : (3 : F) * coeffOf D 1 3 2 n = 0 := by linear_combination hRb - hRa
    exact (mul_eq_zero.mp h3y).resolve_left h3                -- det≠0 ⇒ the coeff is 0
  exact ⟨by rw [hRa]; exact hy, hy⟩
```
Recurring gotchas (all bit this session): `linear_combination` not `linarith` (general
`[Field F]` has no order); resolve `3·y = 0` with `(mul_eq_zero.mp h3y).resolve_left hp`;
keep the `[CharNeTwo F]` instance in the `variable` block even when the linter calls it
"unused" (it's load-bearing for `coeffOf_cond_of`/`bracketIdentity_eq_expanded` synthesis
— the `unusedSectionVars` false-friend); reuse Core `HalfDerivation.bracket_zero`
(commuting ⇒ `bracketIdentity = 0`) instead of re-deriving or importing a downstream
upper-bound module.

## 3. Circularity: a lemma proving P must NOT call a sub-lemma that ASSUMES P

The Gate-3 finding. `imageContainment` ESTABLISHES "image ⊆ I" (i.e. it proves the
predicate `I_filtered`). But its boundary leaves used `width_stability`, whose signature
takes `hI : I_filtered D.coeff` as a hypothesis. **A lemma that proves P cannot lean on a
sub-lemma that assumes P** — that's circular, and an `hI`-free statement simply has no
`hI` to pass.

- **Detect it from the signatures**, before writing the proof: if your target is hI-free
  but a leaf needs hI, you have a circularity (or a vacuous theorem — if `P ≡ "the very
  thing you conclude"`, assuming `hI` makes the theorem trivially true, a red flag).
- **Fix**: internalize the assuming-lemma's CONTENT into the same induction. Its
  hypothesis (`hI`) was only used to kill smaller-measure objects → those come from the
  induction's own `ih` instead. The ONE object that doesn't descend (the same-level
  coupling node, §1–2) gets the explicit coupling lemma. So `width_stability` is
  re-derived hI-free as `{ih + coupling lemma}`.
- This is also WHY the mainline must add `char ≠ p`: the coupling lemma needs `det ≠ 0`.

## Quick checklist
1. Hypothesis-use audit: every assumption, what measure does the object it kills have?
   all strictly-smaller ⇒ ih-replaceable; same/larger ⇒ coupling node.
2. Coupling node ⇒ explicit k×k system; if `det = p`, add `char ≠ p`.
3. hI-free target using an hI-dependent leaf ⇒ circularity ⇒ internalize (ih + coupling).
4. Verify the lemma in `lean_run_code` first; confirm `lean_verify` axioms ⊆
   {propext, Classical.choice, Quot.sound} (no `sorryAx`) and NO `I_filtered`/`hI` in
   the signature (proves it's genuinely assumption-free).
