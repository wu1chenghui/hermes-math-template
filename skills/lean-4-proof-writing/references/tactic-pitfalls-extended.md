# Extended Tactic Pitfalls — Lean 4 Proof Writing

Collected from the 1/2-derivation classification project (2026-06).
These go beyond the basic pitfalls in the main SKILL.md.

---

## `split_ifs with hA hB` — false branches are function types

`split_ifs with hA hB hC hD` creates named hypotheses for each if-condition.
For **true** branches the hypothesis is an inductive `∧`; for **false** branches
it is `¬(cond)` = `cond → False` — a function type.  `rcases` cannot destruct a
function type.

**Fix**: either use `split_ifs` without `with` (and use `‹...›` notation) or
prefer `by_cases` with nested conditions.  The `by_cases` approach is more
reliable for multi-condition case analysis.

---

## `simp` cannot use `¬(P ∧ Q)` to simplify `if P ∧ Q then ... else 0`

`simp` only uses direct `¬` on atomic propositions, not on compound `∧`.  To
simplify `if u = c ∧ d < v then ... else 0` when you know `u ≠ c`:

```lean4
-- ✓ WORKS:
simp [show u ≠ c from by omega]  -- `u = c` becomes `False`

-- ✗ DOES NOT WORK:
simp [show ¬(u = c ∧ d < v) from ...]  -- does NOT simplify the `∧`
```

Provide the **individual negated components**, not the negated conjunction.

---

## `linarith` does not work on field `F`

`linarith` works over `ℕ`, `ℤ`, `ℚ`, `ℝ` but not over arbitrary fields.  Use:

```lean4
-- From (2:F)*0 = -x, derive x = 0:
calc
  x = -(-x) := by ring
  _ = -(0 : F) := by rw [← h_cond, show (2:F)*0 = 0 by ring]
  _ = 0 := by simp
```

Or use `neg_eq_zero.mp` after rewriting to `-x = 0`.

---

## `subst` clears variables; prefer `rw`

`subst h` replaces ALL occurrences of the variable and **removes it from the
context**. If you later reference that variable in a `have` block, it's gone.
Prefer:

```lean4
rw [h]  -- replaces in goal only, variable stays in context
```

---

## Pattern: `by_cases` more reliable than `split_ifs` for 4-condition analysis

For goals with multiple `if`-conditions (16 branches), use nested `by_cases`
with decomposed hypotheses:

```lean4
by_cases hA : u < c ∧ v = d
· rcases hA with ⟨huc, hvd⟩
  simp [huc, hvd, show u ≠ c from by omega]  -- individual components
  by_cases hC : u = i ∧ i+1 < v
  · ...  -- A+C active
  · ...  -- Only A
· by_cases hB : u = c ∧ d < v
  · ...
  · ...
```

This avoids the `split_ifs with` naming traps and `simp` conjunction issues.
