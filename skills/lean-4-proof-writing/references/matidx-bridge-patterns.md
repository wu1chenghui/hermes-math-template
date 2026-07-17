# MatIdx Bridge & half_deriv_cond Patterns

Patterns learned during the Classification/DerivedZero/ImageInCenter Lean proofs.

## MatIdx proof irrelevance: binder → canonical form

When a lemma works with canonical MatIdx `⟨x.i, x.i+1, ...⟩` but you have
a binder `x : MatIdx n`, bridge them with `MatIdx.ext_iff`:

```lean4
-- Given: h_adj : x.j = x.i + 1
have h_canon : (⟨x.i, x.i+1, x.hpos, by omega, by omega⟩ : MatIdx n) = x :=
  (MatIdx.ext_iff _ _).mpr ⟨rfl, h_adj.symm⟩
rw [← h_canon]
-- Now the goal uses the canonical form, matching the lemma's conclusion
```

The `MatIdx.ext_iff` lemma (defined once, shared across files):
```lean4
lemma MatIdx.ext_iff {n : ℕ} (a b : MatIdx n) : a = b ↔ a.i = b.i ∧ a.j = b.j := by
  constructor
  · intro h; subst h; simp
  · intro ⟨hi, hj⟩; cases a; cases b; simp at hi hj; simp [hi, hj]
```

**Key:** `.mpr` with `⟨rfl, h_adj.symm⟩` — note `.symm` on `h_adj` because
`a.j = x.i+1` but `b.j = x.j = x.i+1`, so we need `x.i+1 = x.j` which is
`h_adj.symm`.

**Pitfall:** `cases x; simp` or `dsimp` on MatIdx often fails because
`let` binders in noncomputable definitions block reduction.
`MatIdx.ext_iff` is the reliable alternative.

**Pitfall 2:** `rw` direction matters. Use `rw [← h_canon]` to replace
the canonical form in the GOAL with the binder, or `rw [h_canon]` for
the opposite direction.

## `subst` + `open MatIdx` projection collision

When `open MatIdx` is active, `subst h` can cause field projections
(`.i`, `.j`) to shadow binder variables. Fix: use `rw [h]` instead of
`subst h`.

```lean4
-- ❌ subst h_u_eq_i makes i resolve to MatIdx.i projection
subst h_u_eq_i

-- ✅ rw avoids the collision
rw [h_u_eq_i]
```

## `split_ifs` + `ring` pattern for half_deriv_cond

When proving `half_deriv_cond` for a linear combination like `T = D - c·Id`,
the `calc` + `h_if_distrib` + `rw` + `ring` approach often fails because
`ring` can't handle `dite` binders. Use `split_ifs` instead:

```lean4
-- After rw [hD, hId]: goal is RHS_D - c*RHS_Id = (RHS with D.coeff - c*Id.coeff)
split_ifs
· ring  -- case 1
· ring  -- case 2
...      -- 16 cases total (4 independent dite conditions)
· ring  -- case 16
```

**Why this works:** `split_ifs` resolves all `dite`/`ite` terms to concrete
values in each branch. Each branch becomes a pure ring identity that `ring`
can close directly, without needing manual `h_if_distrib` lemmas.

**Count:** 4 conditions in the half-derivation RHS (IA, IB, IIA, IIB), so
`split_ifs` creates up to 16 subgoals. Some are contradictory and
`omega` closes them.

**Full proof example** (from DerivedZero.lean):
```lean4
have h_half_deriv := by
  intro x y k hk hk'
  dsimp [Tcoeff]
  have hD := D.half_deriv_cond x y k hk hk'
  have hId := (idHalfDeriv F n).half_deriv_cond x y k hk hk'
  have h_expand : (2 : F) * (D.coeff x y - c * Id.coeff x y)
      = (2*D.coeff) - c*(2*Id.coeff) := by ring
  rw [h_expand, hD, hId]
  split_ifs
  · ring; · ring; · ring; · ring
  · ring; · ring; · ring; · ring
  · ring; · ring; · ring; · ring
  · ring; · ring; · ring; · ring
```

## `noncomputable def` and definitional equality

`noncomputable def` in Lean 4 uses `opaque` — `rfl` and `dsimp` do NOT work.
Use `rw` with the equation lemma instead:

```lean4
noncomputable def a_coupled (D : HalfDerivation F n) (hn3 : 3 ≤ n) : F := ...

-- ❌ dsimp doesn't unfold noncomputable defs
dsimp [a_coupled]  -- "made no progress"

-- ✅ rw uses the equation lemma
rw [a_coupled F n D hn3]

-- ✅ unfold works (not dsimp)
unfold a_coupled
```

**Exception:** `let` binders ARE definitional, even inside `noncomputable` defs.
So `let Tcoeff := ...` inside a `noncomputable def T := ...` allows `dsimp [Tcoeff]`.

## `diagCoeff_eq` for arbitrary MatIdx (diagonal case)

To prove `T.coeff x x = 0` for adjacent `x` (x.j = x.i+1):
1. Use `(diagCoeff_eq D x.i (x.i+1) x.hpos (by omega) (by omega)).symm` to bridge
   `D.coeff x x = diagCoeff D x.i (x.i+1)` (the `.symm` is needed because the
   lemma gives the opposite direction)
2. Use `classification_n_ge_5` to get `diagCoeff = scalarCoeff`
3. Use `id_at_diag` to get `Id.coeff x x = 1`
4. Then `T.coeff x x = D.coeff - c*Id.coeff = c - c*1 = 0`

Full proof:
```lean4
theorem T_adjacent_diag_zero_gen (D : HalfDerivation F n) (hn5 : 5 ≤ n) (h2ne : (2 : F) ≠ 0)
    (x : MatIdx n) (h_adj : x.j = x.i + 1) : (T F n D).coeff x x = 0 := by
  rw [T_coeff_eq F n D x x]
  have h_Id : ((idHalfDeriv F n).coeff x x) = (1 : F) := id_at_diag F n x
  rw [h_Id]
  have h_D : D.coeff x x = scalarCoeff F n D := by
    have h_diag := (diagCoeff_eq D x.i (x.i+1) x.hpos (by omega)
      (by rw [← h_adj]; exact x.hle))
    let canonical : MatIdx n := ⟨x.i, x.i+1, x.hpos, by omega, by
      rw [← h_adj]; exact x.hle⟩
    have hx_eq : x = canonical := (MatIdx.ext_iff x canonical).mpr ⟨rfl, h_adj⟩
    rw [hx_eq] at h_diag
    -- h_diag: diagCoeff D x.i (x.i+1) = D.coeff canonical canonical
    rw [← h_diag.symm]
    calc
      D.coeff canonical canonical = diagCoeff D x.i (x.i+1) := h_diag.symm
      _ = (toRecursiveSystem D).C x.i (x.i+1) := rfl
      _ = (toRecursiveSystem D).C 1 2 := classification_n_ge_5 ...
      _ = scalarCoeff F n D := by unfold scalarCoeff; simp [hn5]
  rw [h_D]; ring
```
