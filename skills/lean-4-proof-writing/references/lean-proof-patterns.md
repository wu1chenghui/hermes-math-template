# Proof patterns: split_ifs + ring, MatIdx irrelevance, omega pitfalls

## `split_ifs` + `ring` for linearity proofs

When proving a structure field condition that is linear in the structure's data
(e.g., `half_deriv_cond` for `T = D - c·Id`), the `calc` + `h_if_distrib` + `ring`
approach often fails because `rw` can't match all `dite`/`ite` terms or `ring` can't
handle the resulting expression.

**Instead: use `split_ifs` + `ring`.**

```lean4
-- Goal: (2:F)*(D.coeff x y - c*Id.coeff x y) = (dite_RHS with D.coeff - c*Id.coeff)
rw [h_expand, hD, hId]  -- expand LHS and rewrite using the two known equations
split_ifs               -- break all dite/ite into 16 cases (4 conditions × T/F)
· ring
· ring
...                     -- 16 ring blocks total
```

- `split_ifs` handles BOTH `dite` (with binder `h : cond`) and `ite` uniformly.
- Each branch gets concrete values instead of conditional expressions.
- `ring` closes the resulting algebraic identity trivially.
- More reliable than: `calc` + `repeat (rw [h_if_distrib])` + `ring`.

## MatIdx proof irrelevance bridge

When a lemma's conclusion uses a canonical MatIdx construction (e.g., `⟨i,i+1,...⟩`)
but the goal uses a binder `x : MatIdx n` with the same `.i` and `.j` fields,
they are NOT definitionally equal in Lean. Use `MatIdx.ext_iff`:

```lean4
have h_canon : (⟨x.i, x.i+1, x.hpos, by omega,
    by rw [← h_adj]; exact x.hle⟩ : MatIdx n) = x :=
  (MatIdx.ext_iff _ _).mpr ⟨rfl, h_adj.symm⟩
rw [← h_canon]
-- Goal now uses canonical MatIdx, matching the lemma's conclusion
```

Key points:
- `MatIdx.ext_iff` must be defined: `a = b ↔ a.i = b.i ∧ a.j = b.j`
- The `.mpr` direction needs `(a.i = b.i ∧ a.j = b.j)`.
- `.symm` is needed on `h_adj` because canonical `.j = x.i+1`, and we need `x.i+1 = x.j`.

## Omega pitfall: `omega; omega` / `by omega; omega`

```lean4
-- ❌ Wrong: second omega has no goal
have h_adj : x.j = x.i + 1 := by
  have h_lt : x.j - x.i < 2 := by omega; omega  -- error!

-- ✅ Correct: single omega, separate have blocks
have h_adj : x.j = x.i + 1 := by
  have h_lt : x.j - x.i < 2 := by omega
  omega
```

After `omega` closes the first subgoal, the `;` chains to the next tactic which
finds no remaining goals → "No goals to be solved".

## `apply hy_not_b` vs `exact hy_not_b` on `∧` goals

When the goal is `¬(A ∧ B ∧ C)` and `hy_not_b : ¬(A ∧ B' ∧ C)`, prefer:

```lean4
-- ✅ Reliable: apply + refine with ?_
intro h; rcases h with ⟨ha, hb, hc⟩
apply hy_not_b
refine ⟨ha, ?_, hb, hc⟩
-- now prove the ?_ gap

-- ❌ Unreliable: direct exact with projections
exact fun h => hy_not_b ⟨h.1, ..., h.2.1, h.2.2⟩
-- .2.1/.2.2 projections fail unpredictably with ∧ chains
```

The `apply`+`refine`+`?_` pattern creates an explicit subgoal that's easier to
prove than matching projections in a single `exact`.
