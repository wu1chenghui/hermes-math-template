# Lean 4 `simp` and `if`-condition pitfalls

## `simp` cannot use `¬(P ∧ Q)` to simplify `if P ∧ Q then ...`

`simp` works with DECOMPOSED hypotheses, not compound ones:
- ✅ `simp [huc, hvd]` where `huc : u < c`, `hvd : v = d` — simplifies `if u < c ∧ v = d`
- ❌ `simp [hA]` where `hA : u < c ∧ v = d` — does NOT simplify the if-condition  
- ❌ `simp [hB_false]` where `hB_false : ¬(u = c ∧ d < v)` — does NOT simplify the if-condition

**Fix**: Decompose `∧` hypotheses with `rcases` before using `simp`, and for negated `∧`,
provide individual `¬` components: `simp [show u ≠ c from by omega, show ¬(d < v) from by omega]`.

## Pattern: explicit `if`-condition expansion (recommended)

When `simp` fails on `if`-conditions in a ring expression, use explicit `have` blocks:

```lean4
have hA' : (if u < c ∧ v = d then coeffOf D i (i+1) u c else 0) = coeffOf D i (i+1) u c := by
  simp [huc, hvd]
have hB' : (if u = c ∧ d < v then coeffOf D i (i+1) d v else 0) = 0 := by
  simp [show u ≠ c from by omega]
rw [hA', hB']; simp
```

This is more reliable than trying to `simp` the entire goal at once.

## `linarith` on `Field F` variables

`linarith` works on `ℕ`, `ℤ`, `ℚ`, `ℝ` but NOT on arbitrary `Field F` variables.
Use `ring` or `calc` for field algebra instead:
```lean4
calc
  coeff = -(-coeff) := by ring
  _ = -(0 : F) := by rw [← h_simp, show (2:F)*0 = 0 by ring]
  _ = 0 := by simp
```

## `subst` clears the variable

`subst h` where `h : x = y` eliminates `x` entirely, replacing all occurrences with `y`.
If later proof steps need the original variable name (or `h_second` refers to it), use `rw [h]` instead.

## `split_ifs with` produces `→ False` hypotheses

`split_ifs with hA hB` creates `hA : ¬P` (i.e., `P → False`) for false branches.
`rcases` cannot destructure a function type. 

**Fix**: use `split_ifs` without `with` and use `‹...›` notation, or use `by_cases` on each
condition individually. For multiple conditions, `by_cases` nesting is more verbose but
avoids all these issues.
