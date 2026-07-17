# subst-after-split_ifs pitfall

## Problem

After `split_ifs` in a Lean proof, using `subst` to substitute equalities from
the split_ifs hypotheses (e.g. `subst hvd` for `hvd: v = d`) corrupts the
variable context. When multiple `subst` calls chain, subsequent `by_cases` on
arithmetic comparisons like `by_cases h : j ≤ i + 1` fail with:

```
error: Failed to infer type of binder
error: don't know how to synthesize implicit argument
```

This was discovered in `coeffOf_cond_zero`'s 16-case `split_ifs` proof in
`/opt/lean-home/lean-projects/e/E/Bracket.lean`. Two `subst` calls in sequence
(`subst hvd; subst hui`) broke `by_cases h_adj_ij : j ≤ i + 1`.

## Root Cause

`subst` uses `Eq.rec` to rewrite, which can make the context very complex.
After chained `subst`s, the binder types for new `by_cases` become un-inferrable.

## Fix

Use `rw` on the goal only (not `at *`), and create explicit copies of
hypotheses that reference the variables being rewritten:

```lean4
  rcases ‹u < c ∧ v = d› with ⟨huc, hvd⟩
  rcases ‹u = i ∧ j < v› with ⟨hui, hjv⟩
  -- Copy hypotheses BEFORE rewriting
  have huc' : i < c := by rw [← hui]; exact huc  -- u becomes i
  have hjv' : j < d := by rw [← hvd]; exact hjv  -- v becomes d
  -- Rewrite only the goal
  rw [hvd, hui]
  -- Now use huc', hjv' in downstream calls (coeffOf_nonadjacent, etc.)
```

Note: `rw [hvd, hui] at *` does NOT reliably rewrite all hypotheses either.
The `rw` may skip hypotheses whose types don't match the rewrite pattern.
Explicit copies are more reliable.

## Alternative

Use `generalize` or avoid `split_ifs`'s named hypotheses entirely — use
`by_cases` on atomic conditions instead of compound `∧` conditions.
