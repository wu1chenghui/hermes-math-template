# T3 Pitfall in `bracketIdentity_eq_expanded`

## The pattern

`bracketIdentity_eq_expanded` expands as four terms:
```
T1: (if u < c ∧ v = d then coeff i j u c else 0)    -- source-row < partner-row, target-col = partner-col
T2: -(if u = c ∧ d < v then coeff i j d v else 0)    -- source-row = partner-row, partner-col < target-col
T3: +(if u = i ∧ j < v then coeff c d j v else 0)    -- target-row = source-row, source-col < target-col
T4: -(if u < i ∧ v = j then coeff c d u i else 0)    -- target-row < source-row, target-col = source-col
```

## The pitfall: T3 fires when `j < v`

A common coding error in this project: **assuming T3 never fires** and using `if_neg c3`
with `c3 : ¬(u=i ∧ j<v)` proved by `omega`.  But `omega` can only prove the negation
when `j ≥ v` (source col ≥ target col).  When `j < v` (source col < target col), T3
**does fire**, and `omega` correctly refuses to prove the false negation.

### Where this happened

In `width_a_chain_zero`, bracketIdentity(2,3; n-1,n; 2,n):
- u=2, i=2 → u=i ✓
- j=3, v=n → 3<n for n≥5 ✓
- **T3 fires**: coeff(n-1,n; 3, n)

The original code had `c3 : ¬(2=2 ∧ 3<n) := by omega` which was **logically false**
for n≥5.  `omega` correctly reported it couldn't prove a false statement.

### The fix

1. **Recognise T3 fires**: change `if_neg c3` to `if_pos c3` with `c3 : u=i ∧ j<v`
2. **Extract the T3 term**: it introduces a coefficient on the PARTNER source
3. **Close the partner coefficient separately**: may need a new lemma or a different bracket

In the width_a case, T3 introduces `coeff(n-1,n; 3, n)` (for j'=3) or
`coeff(n-1,n; j', n)` (for general j').  These are partner-side coefficients
that currently remain as sorries.

### When T3 is harmless vs dangerous

- **T3 harmless**: when the T3 coefficient is independently known to be zero
  (e.g., known by a prior lemma, or the partner is outside the active domain)
- **T3 dangerous**: when the T3 coefficient is unknown and couples the proof
  to a new coefficient that may require induction or new machinery

### Detection rule

Before writing `c3 : ¬(u=i ∧ j<v) := by omega`, ask: **is `j < v` always false?**
If `j` and `v` are variables with `j ≤ n` and `v ≤ n`, omega CANNOT guarantee
`j ≥ v` — it depends on the specific branch.  Use `by_cases hj_lt_v : j < v` first,
then handle both cases.
