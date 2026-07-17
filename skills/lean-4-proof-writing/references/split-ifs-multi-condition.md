# split_ifs technique for multi-condition case analysis

## The problem

When a goal contains multiple `if ... then ... else ...` expressions (e.g., 4 conditions creating 16 cases), `split_ifs` is the natural tactic. But naming the hypotheses with `split_ifs with h1 h2 h3 h4` is unreliable — `h2`/`h3`/`h4` may not exist in all branches.

## The solution: `‹...›` notation

Use `split_ifs` with NO `with` clause, then reference hypotheses by TYPE using French quotes:

```lean4
  split_ifs
  · -- Branch: all 4 conditions true
    rcases ‹u < c ∧ v = d› with ⟨huc, hvd⟩
    rcases ‹u = c ∧ d < v› with ⟨huc_eq, _⟩
    omega
  · -- Branch: first 3 true, last false
    rcases ‹u < c ∧ v = d› with ⟨huc, hvd⟩
    rcases ‹u = c ∧ d < v› with ⟨huc_eq, hdv⟩
    -- hdv: d < v is a hypothesis; ¬(u < i ∧ v = j) is present but not named
    ...
```

`‹type›` looks up a hypothesis by its TYPE regardless of auto-generated name. Works for both positive conditions (`‹u = i›`) and conjunctions (`‹u < c ∧ v = d›`).

## After extraction: use `subst`, not `rw`

```lean4
    rcases ‹u = i ∧ j < v› with ⟨hui, hjv⟩
    subst hui    -- ✓ substitutes u → i everywhere
    -- NOT: rw [hui]  — ✗ may fail if split_ifs already simplified the goal
```

`subst` eliminates the variable from all hypotheses AND the goal. `rw` may fail because `split_ifs` sometimes resolves the if-expressions past the point where the variable appears in the goal.

## Real-world example (from E/Bracket.lean)

16-case analysis for `coeffOf_cond_zero` — the 4-term zero-bracket identity:

```lean4
lemma coeffOf_cond_zero (...) :
    (if u < c ∧ v = d then coeffOf D i j u c else 0)
  - (if u = c ∧ d < v then coeffOf D i j d v else 0)
  + (if u = i ∧ j < v then coeffOf D c d j v else 0)
  - (if u < i ∧ v = j then coeffOf D c d u i else 0) = 0 := by
  split_ifs
  · -- T T T T: all 4 conditions true
    rcases ‹u < c ∧ v = d› with ⟨huc, hvd⟩
    rcases ‹u = c ∧ d < v› with ⟨huc_eq, _⟩
    omega
  · -- T T T F
    ...omega...
  ...
  · -- F F F F: all 4 conditions false → all terms 0
    ring
```

9 of the 16 cases are closed by `omega` (index contradictions). 7 require mathematical content using `half_deriv_cond` or `diagonalization`.

## Key insight

`‹...›` notation + `split_ifs` (no `with`) is the ONLY reliable way to do multi-condition `split_ifs` in Lean 4. The `with h1 h2 h3 h4` naming was confirmed broken in practice — hypotheses exist only in branches where their condition is true, making the naming scheme unpredictable.
