# Lean Tactic Pitfalls

## `simp` cannot use `¬(P ∧ Q)` to simplify `if P ∧ Q then A else 0`

`simp` does not decompose `¬(P ∧ Q)` into `¬P ∨ ¬Q`. When you have
`h : ¬(u = c ∧ d < v)` and the goal contains `(if u = c ∧ d < v then ... else 0)`,
`simp [h]` makes no progress. **Provide individual negated components**:

```lean4
  -- FAILS:
  simp [hA] where hA : ¬(u < c ∧ v = d)

  -- WORKS:
  simp [show u ≠ c from by omega, show ¬(d < v) from by omega]
  -- or equivalently, break apart:
  have hu_ne_c : u ≠ c := by omega
  simp [hu_ne_c]
```

The same issue arises with `¬(P ∧ Q)` as a `by_cases` hypothesis:
`simp` treats it as opaque; it needs `P → False` and `Q → False` individually.

## `split_ifs with` creates `→ False` for false branches — prefer `have` + `rw`

`split_ifs with hA hB hC hD` creates hypotheses on ALL branches:
- True branches get `hA : P` (an inductive `∧`)
- False branches get `hA : ¬P` which is `P → False` (a function type)

**`rcases` cannot destruct a function type.** This means any branch where a
condition is false breaks `rcases hA with ...`.

**Reliable alternative**: use `by_cases` + `have hX' : (if ... then coeff ... else 0) = ...` + `rw`:

```lean4
  by_cases hA : u < c ∧ v = d
  · rcases hA with ⟨huc, hvd⟩
    have hA' : (if u < c ∧ v = d then coeff ... u c else 0) = coeff ... u c := by simp [huc, hvd]
    have hB' : (if u = c ∧ d < v then coeff ... d v else 0) = 0 := by simp [show u ≠ c from by omega]
    rw [hA', hB']
    ...
  · have hA' : (if u < c ∧ v = d then coeff ... u c else 0) = 0 := by simp [hA]
    rw [hA']
    ...
```

This pattern handles each if-condition independently, avoids the `False` function
type problem, and makes the proof structured and readable.

## `rename_i` with `split_ifs` is fragile — branch ordering is hard to predict

`split_ifs` on 4 conditions generates 16 branches in a truth-table order.
`rename_i hA hC` renames the **first two** hypotheses in order, regardless
of which condition they correspond to. In a branch where A is false and B is true,
the first hypothesis is `¬A`, not `B`. So `rename_i hB` gets `¬A`, not `B`.

**Avoid `rename_i` with `split_ifs`.** Use `case` syntax or the `have`+`rw` pattern above.

## `linarith` and `nlinarith` do NOT work on `Field F`

`linarith` works on `Nat`, `Int`, `ℚ`, `ℝ` — not on generic `Field F`.
`nlinarith` also fails on fields. For field arithmetic, use `ring` and `calc`:

```lean4
  -- FAILS:
  linarith   -- goal: coeffOf D i (i+1) i v = 0, hyp: (2:F)*0 = -coeffOf ...
  nlinarith  -- same failure

  -- WORKS:
  calc
    coeffOf D i (i+1) i v = -(-(coeffOf D i (i+1) i v)) := by ring
    _ = -((2 : F) * (0 : F)) := by rw [h_cond]
    _ = -(0 : F) := by ring
    _ = 0 := by simp
```

When you have `(2:F)*0 = -x` and need `x = 0`, use `ring` to simplify `(2:F)*0` to `0`,
then `calc` to derive `x = 0` via negation.

## `rfl` fails for associativity mismatches

When two expressions are equal but differ in associativity of `+`/`-`,
`rfl` will NOT work because the terms are not definitionally equal:

```lean4
  -- FAILS: (a - b) + (c - d) vs a - b + c - d
  rfl

  -- WORKS:
  unfold def1 def2; ring
```

Lean parses `a - b + c - d` as `((a - b) + c) - d`, not `(a - b) + (c - d)`.

## `rcases` destroys the hypothesis name

When you `by_cases h : P ∧ Q` and then `rcases h with ⟨hp, hq⟩`,
the original hypothesis `h` is **consumed** — it no longer exists.
Any later `simp [h, ...]` will fail with "Unknown identifier".

**Fix**: use the field names instead:
```lean4
  by_cases h : u < c ∧ v = d
  · rcases h with ⟨huc, hvd⟩
    simp [huc, hvd, coeff_zero]  -- use field names, not h
  · simp [h]  -- h is intact in this branch
```

## Circular dependencies: extract the dependent lemma to a new file

When lemma A (in file X) needs lemma B (in file Y) but Y imports X,
**create a new file Z that imports both X and Y**, and place the lemma there.

This session's example: `coeffOf_cond_zero` (in `Bracket.lean`) used
`coeffOf_nonadjacent` but was needed by `ToNCoeff.lean` which is needed by
`Adjacent.lean`. The fix was `E/BracketCondZero.lean` which imports both
`Bracket.lean` and `Semantics.lean`, breaking the cycle.
