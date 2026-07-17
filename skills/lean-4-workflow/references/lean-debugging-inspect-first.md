# Lean Debugging: Inspect Before Fixing

When `simpa`, `rw`, or any tactic fails with "type mismatch: After simplification",
do NOT iterate on the tactic. Stop and inspect the ACTUAL term.

## The pattern

```lean
-- DON'T do this (iteration without information):
simpa using h
simpa [a] using h
simpa [a, b] using h
simpa [a, b, c] using h
-- ...10 more attempts

-- DO this instead:
-- Option 1: Insert a diagnostic block
have h_diag : True := by
  have : <the goal shape you expect> := by
    simpa using h    -- THIS will fail and show both types
  trivial
-- Read the error message to see:
--   expected: <your goal>
--   actual: <what h simplifies to>

-- Option 2: Use set_option pp.all
set_option pp.all true in
have h_diag : <expected type> := by
  simpa using h
-- Now the error shows fully-expanded terms

-- Option 3: Use #check in lean_run_code
-- isolate the relevant terms and check their types
```

## Why this works

The type mismatch error message shows BOTH the expected type and the actual type
of the term after simplification. This is THE information you need to fix the
problem — more iterations of `simpa` just burn tokens without gaining information.

## Common causes of "type mismatch: After simplification"

1. The term is `2*x + 0 = 0` not `2*x = 0` — needs `simp` or `ring`
2. The term is `0 = 2*x` not `2*x = 0` — needs `eq_comm`
3. `rw` didn't fire because the pattern uses a `set` let-in (see `set-let-in-rw-pitfall.md`)
4. `simp` added `(-1)*x` instead of `-x` — needs `ring` not `simp`
5. The expression has implicit type arguments that `simp` doesn't touch
