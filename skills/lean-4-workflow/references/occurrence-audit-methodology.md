# Occurrence Audit Methodology

> **Principle**: Before asking "how do I prove this goal is zero?", ask "WHERE does this goal
> appear in the half-Leibniz equation system?"  The search space is (source, partner, target,
> T1/T2/T3/T4/bracketSource), NOT just "find a partner."

## The Problem with Local Partner Search

When a coefficient resists closure, the natural reflex is to iterate partner choices.
But the search space is combinatorially large and local search can easily miss occurrences
because:

1. Previous audits often **fix the source** to be the coefficient's own source `(i,i+1)`
2. They **fix bracket_zero** as a prerequisite (commuting partners only)
3. They search for the goal in a **specific position** (e.g., only T1, or only bracketSource)

**L2099 case study**: Through multiple sessions, agents searched for `coeff(i,i+1; 1,v)` in
bracketIdentity equations with target `(1,v)`.  The goal never appeared there — it appears
as **T1 in equations targeting `(1,v+1)`** (partner `(v,v+1)`, T1 fires when `1<v` and
`v+1=v+1`, giving `coeff(source; 1, v) = GOAL`).

**Lesson**: The goal's target column in the bracketIdentity equation can be DIFFERENT from
the goal's own target column.  T1 gives `coeff(source; target_row, partner_row)`, where
`partner_row` is the first index of the partner — not related to the equation's target.

## The Occurrence Audit

Given a goal coefficient `G = coeff(si, sj; tu, tv)`, enumerate ALL legal
`(source(a,b), partner(c,d), target(u,v))` and check where `G` appears:

- **T1**: `coeff(a,b; u,c)` — fires when `u < c ∧ v = d`
- **T2**: `coeff(a,b; d,v)` — fires when `u = c ∧ d < v`
- **T3**: `coeff(c,d; b,v)` — fires when `u = a ∧ b < v`
- **T4**: `coeff(c,d; u,a)` — fires when `u < a ∧ v = b`

Also record whether the source-partner pair **commutes** (`b ≠ c ∧ a ≠ d`), which
determines whether `bracket_zero` (and hence `bracketIdentity = 0`) can be used.

### Python Enumeration Script

```python
# Occurrence Audit: enumerate ALL (source, partner, target) triples
# for goal coefficient G = coeff(si,sj; tu,tv)

n = 7  # representative small n
goal = (si, sj, tu, tv)

results = []
for a in range(1, n+1):
    for b in range(a+1, n+1):
        for c in range(1, n+1):
            for d in range(c+1, n+1):
                for u in range(1, n+1):
                    for v in range(u+1, n+1):
                        # T1: coeff(a,b; u,c), fires when u < c ∧ v = d
                        t1_fires = (u < c) and (v == d)
                        t1_coeff = (a, b, u, c)
                        
                        # T2: coeff(a,b; d,v), fires when u = c ∧ d < v
                        t2_fires = (u == c) and (d < v)
                        t2_coeff = (a, b, d, v)
                        
                        # T3: coeff(c,d; b,v), fires when u = a ∧ b < v
                        t3_fires = (u == a) and (b < v)
                        t3_coeff = (c, d, b, v)
                        
                        # T4: coeff(c,d; u,a), fires when u < a ∧ v = b
                        t4_fires = (u < a) and (v == b)
                        t4_coeff = (c, d, u, a)
                        
                        for name, fires, coeff in [
                            ("T1", t1_fires, t1_coeff),
                            ("T2", t2_fires, t2_coeff),
                            ("T3", t3_fires, t3_coeff),
                            ("T4", t4_fires, t4_coeff)
                        ]:
                            if fires and coeff == goal:
                                commuting = (b != c) and (a != d)
                                results.append({
                                    "term": name, "source": (a,b),
                                    "partner": (c,d), "target": (u,v),
                                    "commuting": commuting
                                })

# Analyze results:
# - commuting occurrences → potential bracket_zero closures
# - non-commuting → bracketSource couples to other coefficients
# - zero occurrences → goal is structurally invisible to bracketIdentity
#   → constraint must come from bracketSource or §D centering
```

## Interpretation Guide

### Commuting occurrences (bracket_zero available)

These are the **best case**.  `bracket_zero` forces `bracketIdentity = 0` directly.
Check the OTHER coefficients in the same equation:
- If they are all zero (or already proven zero by IH/local SCC) → **direct closure**
- If they involve diagonal coefficients of OTHER sources → **§D centering** (all_diag_equal)
- If they involve same-level coefficients → **local SCC** (2×2 or 3×3, combine with coeffOf_cond)
- If they involve wider sources → **width collapse** / induction hypothesis

### Non-commuting occurrences

These involve bracketSource (non-zero bracket result).  The equation links the goal to
the bracketSource coefficient via the half-Leibniz equation.  More complex to close;
usually requires induction.

### Zero occurrences — structural invisibility

If the goal appears in ZERO bracketIdentity positions (neither T1-T4 nor bracketSource),
it is **structurally invisible** to the half-Leibniz equation system.  In this case:
- The constraint MUST come from brackSource → nonadjacent coefficient → wider source → ...
- Or the coefficient belongs to the §D centering layer (diagonal coupling via all_diag_equal)
- Do NOT iterate partner choices — the iteration space is empty

## Boundary Edge Case Study (L2136)

Goal: `coeff(n-1,n; 1,n-2)` (boundary source `(n-1,n)`, first-row target `(1,n-2)`).

Occurrence audit result (n=7): **4 occurrences**
- T1 @ source=(n-1,n), partner=(n-2,n), target=(1,n) — **commuting** ✓
- T4 @ source=(n-2,n), partner=(n-1,n), target=(1,n) — **commuting** ✓
- T1 @ source=(n-1,n), partner=(n-2,n-1), target=(1,n-1) — non-commuting
- T4 @ source=(n-2,n-1), partner=(n-1,n), target=(1,n-1) — non-commuting

The commuting T1 equation + coeffOf_cond forms a 3×3 det=3 system, already proven as
`boundary_coupling_sigma`.  The goal was trivially closeable — previous sessions
missed it because they didn't enumerate full (source, partner, target) space.

## Terminal DAG for Dispatch Sorries (L2076 Case Study)

When a sorrie splits into multiple sub-cases (e.g., by v<n vs v=n), do NOT implement Lean
for each sub-case individually. Instead, draw a **complete terminal DAG** first:

1. For EACH sub-case, trace the bracketIdentity chain to its terminal coefficient
2. Classify the terminal: which EXISTING lemma covers it?
3. Only when EVERY branch has a classified terminal, implement the Lean code
4. The implementation writes itself: each branch is just `bracketIdentity → known_lemma`

### L2076 example (u=i interior source, n=7)

The sorrie `coeff(i,i+1; i,v)=0` with `(i,v)∉I` splits naturally:

**v < n (9/12 sub-cases):**
```
GOAL: coeff(i,i+1; i,v)
  ↓ bracketIdentity(i,i+1; v,v+1; i,v+1) [commuting]
  = -coeff(v,v+1; i+1,v+1)    ← terminal
  ↓ internal_det_coupling_zero
  → 0   (3×3 SCC det=-1 for i≥2; internal_det_step for i=1)
```
All terminals covered by existing lemmas — NO new math needed.

**v = n (3/12 sub-cases):**
```
GOAL: coeff(i,i+1; i,n)
  ↓ bracketSource via partner(i+1,n)  [non-commuting]
  → coeff(i,n; i,n)  [diagonal, wide source]
  → §D centering (all_diag_equal)
```
Genuinely deferred to §D — leaves a clean interface: `have hReduce : goal = diagonal_term := ...; sorry`.

### DAG-first value

Attempting to write Lean without the DAG would have led to:
- Trying `all_diag_equal` for v<n (wrong layer — doesn't couple to diagonals)
- Missing that `internal_det_coupling_zero` already covers the terminals
- Creating new lemmas for what's already solved

With the DAG, the implementation is ~60 lines of pure bracketIdentity wiring, and the v=n
interface is self-documenting.

## Workflow Discipline

1. **When a coefficient resists closure after 2-3 partner attempts → STOP and do an occurrence audit first.**
2. Run the Python enumeration for n=7 or n=8 (representative small values).
3. Classify occurrences: commuting vs non-commuting, coupling targets (diagonal? same-level? wider?).
4. **When the sorrie has multiple sub-cases, draw the terminal DAG before writing Lean.** Map every branch to an existing lemma or a deferred interface.
5. Only THEN decide: direct closure / local SCC / width collapse / defer to §D.
6. **Never** start designing new induction predicates or rewriting the skeleton before knowing the occurrence landscape.
