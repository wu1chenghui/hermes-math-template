# `set` let-in gotcha: `rw` cannot find the pattern

## Problem

```lean
set m := k + 1 with hm
-- ... many lines later ...
have hm_val : m = 2 := by unfold m; omega
rw [hm_val] at hcond
-- ERROR: Tactic `rewrite` failed: Did not find an occurrence of the pattern
```

**Root cause**: `set m := k+1` creates a LET-IN binder. `rw` with an equality
about `m` cannot match the let-in. `dsimp [m]` unfolds it but may leave
`1+1` un-normalized (not `2`).

## Solutions

### Solution A: Use `rw` with the `hm` hypothesis from `set`

```lean
set m := k + 1 with hm
-- ... later ...
rw [hm] at hcond     -- replaces m with k+1 (the let-in body)
-- If k has been rewritten to 1:
rw [hk1] at hcond    -- k→1, now hm body is 1+1
simp at hcond         -- normalizes 1+1→2, simplifies if-conditions
```

Key insight: `rw [hm]` replaces the let-in NAME with its BODY (`k+1`),
which `rw` CAN find. Then `simp` normalizes arithmetic.

### Solution B: Avoid `set`; use explicit expression

```lean
-- Instead of:
set m := k + 1 with hm
-- Use k+1 directly everywhere, or:
let m := k + 1
-- (let is different from set — use with care)
```

### Solution C: Reconstruct the equation with explicit indices

If the expression is too tangled with let-ins, just re-call the lemma
with explicit numeric indices rather than trying to transform the existing
hypothesis:

```lean
-- Instead of trying to rewrite hcond (which has let-ins):
rw [hk1, hm_val] at hcond  -- fails for let-in

-- Just re-call with explicit indices:
have hfresh := coeffOf_cond_of D 1 j' 2 n ... 2 ...
simp [hj'_lt_n] at hfresh
-- hfresh is clean, no let-ins
```

This is the most robust approach when the existing hypothesis carries
`set` baggage from an outer proof context.
