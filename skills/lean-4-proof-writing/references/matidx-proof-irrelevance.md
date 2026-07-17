# MatIdx Proof Irrelevance

## Problem

`MatIdx n` is a structure with `(i j : ℕ)` and proof fields `(hpos hlt hle)`.
Since proof fields are in `Prop`, two MatIdx values with the same `i,j` should
be equal by proof irrelevance. But Lean does NOT treat them as definitionally
equal — a binder `x : MatIdx n` and a canonical construction
`⟨x.i, x.j, hi, hij, hjn⟩` are syntactically different.

## Symptom

- `rfl` fails to prove `x = ⟨x.i, x.j, ...⟩`
- `cases x; simp` or `cases x; rfl` fails with "unsolved goals" or "made no progress"
- `rw` of a lemma that uses canonical MatIdx on a binder fails with "did not find occurrence"

## Reliable fix: MatIdx.ext_iff

```lean4
lemma MatIdx.ext_iff {n : ℕ} (a b : MatIdx n) : a = b ↔ a.i = b.i ∧ a.j = b.j := by
  constructor
  · intro h; subst h; simp
  · intro ⟨hi, hj⟩; cases a; cases b; simp at hi hj; simp [hi, hj]

-- Usage (when j is known):
have hx : x = (⟨x.i, x.j, hi, hij, hjn⟩ : MatIdx n) := by
  apply (MatIdx.ext_iff _ _).mpr; exact ⟨rfl, rfl⟩
```

## Pattern: bridging binder to canonical for lemma application

When a lemma (like `diagCoeff_eq`) gives an equality with CANONICAL MatIdx, 
but you need it for a binder `x`:

```lean4
-- diagCoeff_eq gives: diagCoeff D i j = D.coeff ⟨i,j,hi,hij,hjn⟩ ⟨i,j,hi,hij,hjn⟩
-- Goal: D.coeff x x = scalarCoeff

-- Step 1: apply diagCoeff_eq at the canonical form
have h_canonical : D.coeff ⟨x.i, x.j, ...⟩ ⟨x.i, x.j, ...⟩ = diagCoeff D x.i x.j := 
  (diagCoeff_eq D x.i x.j ...).symm

-- Step 2: prove x = canonical via MatIdx.ext_iff
have hx : x = ⟨x.i, x.j, ...⟩ := (MatIdx.ext_iff _ _).mpr ⟨rfl, rfl⟩

-- Step 3: rewrite
rw [hx] at goal  -- now goal is about canonical form
-- apply h_canonical and continue
```

## Pattern: T_image_in_center packaging

The classification theorem `T_image_in_center` requires proving vanishing for
arbitrary `x y : MatIdx n`. The sub-lemmas (`T_nonadjacent_zero`, 
`T_adjacent_vanish`, `T_adjacent_diag_zero_gen`) each handle specific cases:

- `T_nonadjacent_zero`: works directly with binders (no canonical issue)
- `T_adjacent_diag_zero_gen`: avoids canonical entirely by using `T_coeff_eq` + 
  `diagCoeff_eq` + `classification_n_ge_5` on ℕ indices
- `T_adjacent_vanish`: uses `T_support_adjacent` which uses canonical `⟨i,i+1,...⟩`

The remaining gap: `T_adjacent_vanish` gives `T.coeff ⟨i,i+1,...⟩ y = 0` but we
need `T.coeff x y = 0` where `x` has `x.i = i, x.j = i+1`. Bridging requires
MatIdx proof irrelevance.

**Current status:** Mathematical content complete. MatIdx irrelevance gap tracked
as Lean engineering issue, not mathematical gap.
