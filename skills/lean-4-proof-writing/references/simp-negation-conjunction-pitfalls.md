# `simp` / `linarith` / `calc` / `subst` pitfalls with negation hypotheses and conjunctions

Patterns from a session on upper-bound spanning of `kerΦ` for N_n
half-derivations (`E/Classification/Spanning.lean`). The common theme:
**rewriting `if` guards that are conjunctions, where only one conjunct's
negation is known**.

## Pitfall 1: `simp` with `¬(a=b)` does NOT simplify `if (a=b ∧ ...)`

The target:  
`(if a = b ∧ c = d then v else 0) = (if a = b' ∧ c = d' then v' else 0)`

Given `h : ¬(a = b)` (a negation hypothesis), `simp [h]` does **nothing** to
the `if (a = b ∧ c = d)`.  Reason: `simp`'s `if_neg` lemma matches only the
top-level `if` condition `(a = b ∧ c = d)` as a whole — it does not break the
conjunction into conjuncts.  `h : ¬(a = b)` does not prove `¬(a = b ∧ c = d)`,
because `c = d` could still be `False`.  So `simp` marks the hypothesis as
unused.

### ❌ Bad (6+ iteration cycles to discover):
```lean
simp [h]  -- warning: `h` is unused by simp; goal unchanged
```

### ✅ Good — explicit rewrite of each term:
```lean
-- For `if (a = b ∧ c = d) then v else 0`, with `h : ¬(a = b)`:
  have hT : (if a = b ∧ c = d then v else 0) = 0 := by
    simp [h]
  rw [hT]
```

Wait — even `simp [h]` fails for the isolated term, for the same reason.
The correct fix:

```lean
  have hT : (if a = b ∧ c = d then v else 0) = 0 := by
    have h' : ¬(a = b ∧ c = d) := by
      intro hc; exact h hc.1
    simp [h']
```

Or shorter:
```lean
  have hT : (if a = b ∧ c = d then v else 0) = 0 := by
    simp [show ¬(a = b ∧ c = d) from fun hc => h hc.1]
```

Or, if you can accept `split_ifs`:
```lean
    split_ifs with hif
    · exfalso; exact h hif.1
    · rfl
```

### General principle

`simp` with `h : ¬P` only rewrites `if P then ... ` when `P` is an **atomic**
proposition. For a conjunction `(P ∧ Q)`, you must either:
- Provide `h_and : ¬(P ∧ Q)` (by `intro hc; exact h hc.1`), OR
- Use `split_ifs` then `exfalso`.

### Bonus trap: `simp`'s `ite_self` eats the `if` before `split_ifs`

If one branch of the `if` simplifies to 0 (e.g. `coeff = 0` by `h_noI`), `simp`
reduces `(if p then 0 else 0)` to `0` using `ite_self`. Then `split_ifs` on the
goal finds NO `if` and fails with `"no if-then-else conditions to split"`.

**Fix**: compute the LHS and RHS separately, then `calc`:
```lean
have hRHS : (if c = n - 1 ∧ d = n then coeff c d 1 (n - 1) else 0) = 0 := by
  simp [hi_n1]
calc
  (if 1 < i ∧ n = j then coeff c d 1 i else 0) = 0 := by simp [h_noI]
  _ = (if i = n - 1 ∧ j = n then coeff c d 1 (n - 1) else 0) := by
    symm; exact hRHS
```
This avoids the `split_ifs` trap entirely.

## Pitfall 2: `linarith` does not handle `neg` (`-x = 0 → x = 0`)

```lean
h : -x = 0
⊢ x = 0
```

`linarith` **fails** with "linarith failed to find a contradiction".  `linarith`
works over commutative semirings and treats `-x` as `0 - x` — this is additive
group theory, not linear arithmetic.

### ✅ Correct:
```lean
neg_eq_zero.mp h
```

Or:
```lean
calc x = -(-x) := by simp; _ = -(0 : F) := by rw [h]; _ = 0 := by simp
```

## Pitfall 3: `calc` cannot chain mixed `<` and `=`

```lean
calc i < j := hij; _ = n := hn_eq.symm   -- ERROR
```

`calc` requires every relation to be the **same** relation.  Mixing `<` with
`=` fails.

### ✅ Correct:
```lean
have hi_lt_n : i < n := lt_of_lt_of_eq hij hn_eq.symm
```
or
```lean
have hi_lt_n : i < n := by
  rw [← hn_eq]
  exact hij
```
or just
```lean
have hi_lt_n : i < n := by omega
```

## Pitfall 4: `subst` can fail with "invalid equality proof"

```lean
h : x = y
-- x and y are both ℕ variables
subst h   -- ERROR: "invalid equality proof, it is not of the form (x = t) or (t = x)"
```

This happens when `x` appears in **other hypotheses** that prevent `subst`
from cleanly eliminating it (e.g. `hij : i < j` and `hjn : j = n` — `subst hjn`
can't eliminate `j` because `hij` depends on it).  Some versions of Lean 4.31
are stricter than others.

### ✅ Correct (choose one):
```lean
rw [hjn] at hij      -- rewire hij: hij becomes i < n
```
or
```lean
apply hij.trans_eq   -- hij.trans_eq hjn.symm gives i < n
```
or
```lean
-- Just avoid subst; use explicit rewrites
rw [hjn, hin1]
```
