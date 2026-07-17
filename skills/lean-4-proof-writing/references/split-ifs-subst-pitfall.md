# Pitfall: split_ifs + subst breaks by_cases in Lean 4

## Symptom

After `split_ifs` + `rcases` + multiple `subst` calls in the same branch,
`by_cases h : j - i ≥ 2` fails with:
```
Failed to infer type of binder `h`
don't know how to synthesize implicit argument `α`
don't know how to synthesize implicit argument `β`
don't know how to synthesize implicit argument `γ`
```

This is a Lean 4 issue with `split_ifs` + multiple `subst`s on variables
that are also binder parameters. The `Eq.rec` from `subst` corrupts the
context in a way that `by_cases` can't resolve.

## Trigger

```lean4
split_ifs
· ...
· rcases ‹u < c ∧ v = d› with ⟨huc, hvd⟩
  rcases ‹u = i ∧ j < v› with ⟨hui, hjv⟩
  subst hvd; subst hui          -- ← TWO substs on binder variables
  by_cases h : j - i ≥ 2        -- ← FAILS here
```

Single `subst` calls (on just one variable) usually work fine.

## Fix: explicit copy + goal-only rewrite

```lean4
rcases ‹u < c ∧ v = d› with ⟨huc, hvd⟩
rcases ‹u = i ∧ j < v› with ⟨hui, hjv⟩
-- Create explicit copies BEFORE rewriting
have huc' : i < c := by rw [← hui]; exact huc
have hjv' : j < d := by rw [← hvd]; exact hjv
rw [hvd, hui]  -- rewrite GOAL only
by_cases h_adj : j ≤ i + 1     -- now works
· sorry
· have h_ge2 : j - i ≥ 2 := by omega
  ...
```

## Alternative: rw at * (less reliable)

```lean4
rw [hvd, hui] at *
```

This sometimes works but may skip hypotheses bound by `rcases` pattern
matches. When it fails, the copy approach above is more robust.

## Preferred by_cases pattern

For checking `j - i ≥ 2`, use:

```lean4
by_cases h_adj : j ≤ i + 1
· -- adjacent: j = i+1 (since i < j from context)
  sorry
· have h_ge2 : j - i ≥ 2 := by omega
  ...
```

This avoids the `j - i` subtraction entirely in the `by_cases` condition,
which is more robust in contexts with complex binder setups.

## Architecture note for ½-derivation projects

When building a coefficient calculus layer (like `Bracket.lean`) that sits
BELOW the classification layer:

1. **Prove what you can with only `half_deriv_cond`** — the nonadjacent
   source diagonalization (`coeffOf_nonadjacent`) can be proven using only
   `coeffOf_cond` (which is derived from `half_deriv_cond`), without needing
   any classification theorems.

2. **Leave adjacent-source cases as `sorry`** — they require the
   classification infrastructure (`T_adjacent_vanish`) which lives in a
   higher layer. Don't try to prove them in the coefficient layer to avoid
   circular dependencies.

3. **For pair-cancellation cases** (A+C, A+D, B+C, B+D): when both sources
   are nonadjacent, each term is individually 0 by diagonalization
   (`coeffOf_nonadjacent`), so their sum/difference is trivially 0. The
   adjacent-source subcases remain as gaps.

4. **Induction structure for diagonalization**: Induct on TARGET length
   (`v-u`), not source length (`j-i`). Generalize `i j u v` in the induction.
   Check `j-i ≥ 2` via `by_cases` INSIDE the lemma (not as a premise), so
   recursive calls re-check the condition. This mirrors `NCoeff.nonadjacent`.

5. **Upgrade path — Semantics Bridge**: When a lemma has ≥8 `split_ifs`
   branches, consider replacing the entire case analysis with a semantics layer.
   Define `bracketLeft`/`bracketRight` as named coefficient extractions
   (A−B and C−D), prove the operator identity ONCE, then project.
   See `references/semantics-bridge-pattern.md` for the full pattern.
