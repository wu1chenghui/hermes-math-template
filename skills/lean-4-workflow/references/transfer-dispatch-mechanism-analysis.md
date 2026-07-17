# TRANSFER Dispatch — Mechanism Analysis (2026-06-27)

## Context

The D3 `imageContainment_skeleton` has a 5-way dispatch on adjacent sources.
The TRANSFER branch handles `u < i` with `v ≠ i+1`:

```
Goal: coeffOf D i (i+1) u v = 0
Where: u < i, v ≠ i+1, (u,v) ∉ I
```

## Core finding: GOAL is structurally invisible to bracketIdentity

**Fact.** For adjacent source `(i,i+1)` and any valid partner `(c,d)` with `c < d`:

- **T1** = `coeff(i,i+1; u, c)` when `u < c ∧ v = d`.  Target column = c.
  T1 matches GOAL only if `c = v`, requiring partner `(v, d)`.  But then
  T1 fires when `v = d`, i.e. partner `(v, v)` — degenerate, invalid.

- **T2** = `coeff(i,i+1; d, v)` when `u = c ∧ d < v`.  Target row = d.
  T2 matches GOAL only if `d = u`, requiring partner `(c, u)` with `c < u`.
  Then T2 fires when `u = c`, i.e. partner `(u, u)` — degenerate, invalid.

- **T3** = `coeff(c,d; i+1, v)` — different SOURCE, never GOAL.
- **T4** = `coeff(c,d; u, i)` — different SOURCE, never GOAL.

**Conclusion:** No legal partner produces bracketIdentity involving GOAL.
The constraint must come from **bracketSource** → nonadjacent coefficient →
`coeffOf_source_exists` + IH — a two-step chain, not a single equation.

## Attempted partner (i+1, v) → partial equation

For `v > i+1`:
- bracket = E_{i,v}
- bracketSource = coeff(i,v; u,v)       (nonadjacent source)
- T1 fires: coeff(i,i+1; u, i+1)       (NOT GOAL)
- Equation: 2·coeff(i,v; u,v) = coeff(i,i+1; u, i+1)

This links coeff(i,v; u,v) to coeff(i,i+1; u, i+1) — the latter is
dispatched via `internal_det_coupling_zero` when `(u,i+1) ∉ I`.

But it does NOT constrain GOAL (= coeff(i,i+1; u,v)).  A **second**
half-Leibniz equation is needed to link GOAL to these known-zero quantities.

## What's needed

A partner pair `(P1, P2)` such that:
- Eq1 relates coeff(i,i+1; u, i+1) to some nonadjacent W
- Eq2 relates GOAL to W (or to coeff(i,i+1; u, i+1))

OR: use `coeffOf_cond_of` on a nonadjacent source (like `(v,i+1)`) to
express GOAL in terms of children with smaller target-width → IH.

The `v ≤ i` subcase has different partner geometry (target column ≤ source
row), requiring separate analysis.

## Attempted partners (all failed to yield GOAL)

| Partner | bracket | bracketIdentity fires | Gives GOAL? |
|---------|---------|----------------------|-------------|
| (i+1,v) | E_{i,v} | T1 only | No (different target col) |
| (v,i+1) | 0 | T2 (u=v, v>i+1) | No (different target row) |
| (u,v)   | 0 (unless v=i) | varies | No (T4 gives different source) |
| (1,2)   | 0 (unless i=2) | T2 or T4 | No (specific u values only) |
| (v,v+1) | 0 | all zero | Trivial (0=0) |

For `v = i` specifically: partner (u,i) gives bracket = -E_{u,i+1}, which
yields coeff(u,i+1; u,i) = 0 — a different coefficient, not GOAL.
