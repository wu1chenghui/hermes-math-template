# First-Row Family Structural Audit

Systematic methodology for diagnosing coefficient invisibility when all standard
approaches (source-shift, partner audit, local SCC, bracket_zero) fail.

## When to use

- A coefficient resists closure after 3+ rounds of partner enumeration
- GOAL INVISIBLE proven for bracketIdentity T1-T4 with source's own partners
- Source-shift, local SCC, and single-step coeffOf_cond all fail

## The methodology

### Step 1: Structural enumeration (NOT partner hunting)

Fix the structural invariant — e.g. target row = 1. Enumerate ALL half-Leibniz
equations involving the family using Python (no Lean type issues):

```
for (a,b) in sources:
    for (c,d) in sources:
        for (u,v) in targets:
            check T1-T4 for GOAL appearance
```

**Critical insight**: The T1 term gives `coeff(src; target_row, partner_row)`.
The equation's TARGET column can be DIFFERENT from GOAL's target column.
Search across ALL target columns, not just the one matching GOAL.

Example: `coeff(2,3; 1,4)` appears as T1 in bracketIdentity with
source=(2,3), partner=(4,5), target=(1,**5**) — note target column is 5, not 4.

### Step 2: Classify visibility

For each non-I target in the family:
- VISIBLE: appears in >=1 equation → identify mechanism (T1/T2/T3/T4)
- INVISIBLE: 0 equations → unreachable via single-step bracketIdentity

### Step 3: INVISIBLE targets

Options when targets are invisible:
1. Multi-step chain via coeffOf_cond + IH (nonadjacent source reduction)
2. May be a case-split artifact (check: does paper need this coefficient?)
3. Defer to §D centering (all_diag_equal → diagonal relation)

### Step 4: VISIBLE targets — target-column partner pattern

The mechanism is typically T1 with partner whose `partner_row` = GOAL's target column:

```
partner = (v, v+1), target = (1, v+1):
  T1: 1 < v ∧ (v+1)=(v+1) → coeff(src; 1, v) = GOAL
  bracketSource = 0 (commuting when src_col≠v and src_row≠v+1)
  T2-T4 dead → bracketIdentity = GOAL → 2*0 = GOAL → GOAL = 0
```

Degenerate cases:
- `v = src_row`: partner=source → T1=T4 tautology. Use (src_row, src_row+2)
- `src_row = v+1`: partner doesn't commute. Use (v, v+2) or (v, src_row+2)

## Key pitfalls

1. **GOAL appears as T1 with DIFFERENT target column.** Earlier "invisibility"
   proofs searched for GOAL where equation's target = GOAL's target. T1 gives
   coeff at (target_row, partner_row), independent of the equation's target column.

2. **bracketSource is NOT coeff(i,j; u,c).** Definition from BracketFormula.lean:
   `(if j=c then coeff(i,d;u,v)) - (if i=d then coeff(c,j;u,v))`.
   Does NOT equal coeff(src; target_row, partner_row). Verify j=c, i=d conditions.

3. **`simp [I_target_set hn]` fails** — `I_target_set hn` is a `Set`, not a
   propositional rewrite. Use: `apply h_notI; rw [heq]; simp [I_target_set]`

4. **`subst` pollutes type inference** with `MatIdx` in deep Lean contexts.
   Prefer `rw [heq]` over `subst heq` when binder types may be ambiguous.

## Reference: L2099 closure

GOAL = `coeff(i,i+1; 1, v)` with `i>=2`, `v!=i+1`, `(1,v) not in I`:

Mini CE experiment (Python enumeration for N_6,N_7,N_8) → ALL non-I targets VISIBLE.

Three sub-cases for partner choice:
- `v = i`:      partner (i, i+2),   target (1, i+2)
- `v = i-1`:    partner (i-1, i+2), target (1, i+2)  [interior only]
- otherwise:    partner (v, v+1),   target (1, v+1)

Boundary case `i+1=n` (source is (n-1,n)) is separate, covered by orphaned
boundary_family_right lemmas.
