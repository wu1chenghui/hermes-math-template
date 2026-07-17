# T3 Cross-Source Elimination Pattern

**Discovered 2026-06-27.** When a coefficient `coeff(i,i+1; u,v)` is structurally
invisible to *all* half-Leibniz equations for source `(i,i+1)` — i.e., appears
in neither bracketSource nor any T1-T4 for any valid partner — try half-Leibniz
for a **different** source where the coefficient appears as T3.

## The pattern

GOAL = `coeff(i, i+1; target_row, n)` with `target_row` > i, `n` rightmost column.

Instead of searching for a partner for source `(i,i+1)` (all fail — GOAL invisible):

1. Use source `(1, target_row)`, partner `(i, i+1)`, target `(1, n)`
2. Bracket = 0 (commuting: `target_row ≠ i` and `1 ≠ i+1` for i ≥ 1)
3. `bracket_zero` → `bracketIdentity = 0`
4. Expand: T3 fires (`u=1=i₁, j<v`) → `coeff(i, i+1; target_row, n)` = GOAL
5. T1, T2, T4 all false (omega)
6. Result: GOAL = 0. No bracketSource, no char≠2, no IH needed.

## Requirements

- `target_row < n` (for valid target)
- `target_row ≠ i` and `1 ≠ i+1` (for bracket_zero commuting condition)
  Both always true for i ≥ 1: target_row > i ensures first; i+1 ≥ 2 > 1 ensures second.

## Concrete instances

- L1830: i, target_row = i+1 → source (1, i+1), partner (i, i+1)
- L1870: i, target_row = i+2 → source (1, i+2), partner (i, i+1)

## Debugging tip

When `bracketIdentity_eq_expanded` rewriting doesn't match, print the RHS with
a temporary `simp` to see the exact `if`-condition text. The pattern:
```
T1: u < c ∧ v = d
T2: u = c ∧ d < v
T3: u = i₁ ∧ j₁ < v    ← where (i₁,j₁) is source 1, NOT original source!
T4: u < i₁ ∧ v = j₁
```
This is `bracketIdentity coeff i₁ j₁ c d u v` — the T3 condition uses source 1's
indices (`i₁`, `j₁`), not the original source's.

## Pitfall: i=1 makes source and partner identical

When i=1 and target_row = i+1 = 2, the source (1,2) and partner (1,2) are
**the same bracket element**. `bracket_zero` still holds ([E_12, E_12] = 0)
but the bracketIdentity expansion degenerates: T2 (`u=c ∧ d<v`) and T3
(`u=i₁ ∧ j₁<v`) BOTH fire, giving `-coeff(i,j; d,v) + coeff(c,d; j,v)` =
`-goal + goal = 0`.  This is a tautology, not a constraint on GOAL.

**Fix**: split on `i=1` before applying the source-shift:
- u = i+1, i=1: target (2,n) ∈ I → `h_notI` contradiction
- u = i+2, i=1: target (3,n) → use `boundary_coupling` lemma directly
- i ≥ 2: source-shift works (source ≠ partner)

## Extended pattern: TRANSFER (u < i, v ≠ i+1)

For GOAL = `coeff(i,i+1; u,v)` where u < i, u ≥ 2, v ≠ i+1:
- Source = (1, u), partner = (i, i+1), target = (1, v)
- Commuting: (i+1) ≠ u (u < i) and 1 ≠ i+1 ✓
- T3 fires (`u_target=1=i_src`, `j_src=u < v`) → GOAL
- bracket_zero → GOAL = 0
- u=1 sub-case: source (1,1) invalid; first-row residue, deferred

## Pitfall: omega can't exclude i=1

`have c2 : ¬(1 = i ∧ ...) := by omega` fails because i=1 IS possible.
When the proof assumes i ≥ 2 but the caller doesn't enforce it,
always split with `by_cases hi1 : i = 1` before the source-shift block.

## Pitfall: h_notI after rw [hv_n]

`rw [hv_n]` rewrites v→n in the goal but NOT in hypotheses.
`_h_notI : (u, v) ∉ I_target_set hn` still references v.
When using h_notI with I_target_set, explicitly `rw [hv_n]` or use
`apply _h_notI; rw [hv_n]` to align types.

## Original pitfall

The original source `(i,i+1)` appears as the **partner** `(c,d)` = `(i, i+1)`.
T3 = `coeff(c,d; j₁, v)` = `coeff(i, i+1; j₁, n)`. For this to be GOAL,
we need `j₁ = target_row`. That's why source 1 = `(1, target_row)`.
