# Simplifying coeffOf_cond Equations — Explicit Condition Pattern

## Problem

`coeffOf_cond` expands into 4 nested `if` expressions. Using `split_ifs` creates
16 branches, most of which are impossible given the index hypotheses. Using
`simp` with `omega` sometimes fails because `omega` can't handle complex
constraint sets inside `simp` blocks.

## Solution: Explicit `have` + `simp`

For each if-condition, write a `have` block that proves it's true or false:

```lean4
-- For source E(i,i+N), split k, projection (u,v)
-- coeffOf_cond gives: (2)*coeff(i,i+N;u,v) = A - B - D + C

-- Step 1: Determine which conditions are true/false
have h_A_false : ¬(v = i+N ∧ u < k) := by
  intro h; rcases h with ⟨hveq, _⟩; omega
have h_C_true : u = i ∧ k < v := ⟨rfl, h_k_lt_v⟩

-- Step 2: One simp call with ALL conditions
simp [h_A_false, h_B_false, h_D_false, h_C_true] at hsplit

-- hsplit now simplifies to the target equality
```

## Why this beats split_ifs

- `split_ifs` generates 2^4 = 16 subgoals, most trivially impossible
- The `have`+`simp` pattern discharges all impossibilities in ONE step
- `omega` works reliably in `have` blocks (simple context), not inside `simp`
  (complex context with many hypotheses)

## Why this beats raw simp with omega

- `simp [show ¬(v=i+N) from by omega]` — omega inside simp blocks sees many
  hypotheses and sometimes fails with "omega could not prove the goal"
- Moving `omega` outside `simp` into standalone `have` blocks gives it a
  cleaner context where it reliably succeeds

## Common Index Relationships

| Condition | When True | When False |
|-----------|-----------|------------|
| `v = i+N ∧ u < k` | u < i+N, u < k, v=i+N | v ≠ i+N or u ≥ k |
| `u = k ∧ i+N < v` | u = k, v > i+N | u ≠ k or v ≤ i+N |
| `v = k ∧ u < i` | v = k, u < i | v ≠ k or u ≥ i |
| `u = i ∧ k < v` | u = i, k < v | u ≠ i or v ≤ k |

Use `omega` to close the trivial inequalities: `by intro h; rcases h with ⟨h1,h2⟩; omega`.
