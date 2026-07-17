# 2×2 SCC via coeffOf_cond C-term

> **Pattern**: When bracketIdentity gives F(c) = F(k) for all c,k (one independent equation),
> coeffOf_cond provides the second equation: 2·F(c) = F(k).
> Combined: 2·F(c) = F(c) → F(c) = 0.

## Anatomy

For a nonadjacent source `(c,u)` with target `(c, v)` where v ≠ u
(typically v = n-1 or some large column):

**coeffOf_cond(c,u; c, v) at split k** (c < k < u):
- **A**: coeff(c,k; c,k) — fires only if v=k.  Usually v ≠ k.
- **B**: coeff(c,k; k,v) — fires only if c=k.  Never (c < k).
- **C**: coeff(k,u; k,v) — fires when **target_row = source_row = c** and k < v.
  Since target_row = c = source_row, and k < v almost always holds,
  **C-term always fires** for this target shape.
- **D**: coeff(k,u; c,k) — fires only if v=k.  Usually v ≠ k.

**Key insight**: For target (c, v) where v ≠ c+1 (i.e., the target column is
far from the source column), ONLY the C-term survives.  All A/B/D terms have
`v=k` conditions that fail.

Result: `2·coeff(c,u; c,v) = coeff(k,u; k,v)` for ANY split k between c and u.

## The full 2×2 system

Combined with bracketIdentity:
1. bracketIdentity(n-1,n; c,u; c,n) → GOAL + coeff(c,u; c,n-1) = 0
   (All partners c<u give the SAME relation — only 1 independent equation)
2. coeffOf_cond(c,u; c,n-1) at split k → 2·coeff(c,u; c,n-1) = coeff(k,u; k,n-1)

With (1) for c and k: F(c) = F(k) = -GOAL.
With (2): 2·F(c) = F(k).
→ 2·F(c) = F(c) → F(c) = 0 → GOAL = 0.

**2×2 system**: variables {F(c), F(k)}, equations {x=y, 2x=y}.  det=1.

## When to use

- Nonadjacent source (width ≥ 2) so coeffOf_cond has a valid split point
- Target row = source row (so C-term fires)
- Target column far enough from source column that A/D terms die (v ≠ split_k)
- bracketIdentity gives the "all partners equal" relation

## When NOT to use

- Adjacent sources (no split point — coeffOf_cond doesn't apply)
- Target column = source column + 1 (A/D may fire for certain splits)
- When bracketIdentity gives genuinely independent equations (3×3 SCC possible)

## Contrast with 3×3 SCC

| | 3×3 SCC (e.g. internal_det_coupling) | 2×2 SCC (this pattern) |
|---|---|---|
| Equations | 2 bracketIdentity + 1 coeffOf_cond | 1 bracketIdentity + 1 coeffOf_cond |
| Variables | 3 | 2 |
| Partner selection | Need TWO distinct commuting partners | Only ONE partner needed |
| coeffOf_cond role | Third equation (det-dependent) | Second equation (always det=1) |
| When to use | Multiple independent bracketIdentity exist | Only 1 independent bracketIdentity |

## Concrete example: boundary_family_right (u≥4)

GOAL = coeff(n-1,n; u,n), u ≥ 4.

- **bracketIdentity(n-1,n; 1,u; 1,n)**: T2 fires (-GOAL), T4 fires (-coeff(1,u;1,n-1)). Eq: GOAL + F(1) = 0.
- **bracketIdentity(n-1,n; u-1,u; u-1,n)**: T2 fires (-GOAL), T4 fires (-coeff(u-1,u;u-1,n-1)). Eq: GOAL + F(u-1) = 0.
  → F(1) = F(u-1) = -GOAL.
- **coeffOf_cond(1,u; 1,n-1) at u-1**: Only C fires (1=1, u-1<n-1). 2·F(1) = F(u-1).

Solve: 2·F(1) = F(1) → F(1) = 0 → GOAL = 0.

**Proof size**: ~90 lines of Lean (bracketIdentity expansion + coeffOf_cond case split).
**No hI, no IH, no width recursion, char≠2 only.**
