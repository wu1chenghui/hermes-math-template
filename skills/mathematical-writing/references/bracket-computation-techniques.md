# Bracket Computation Techniques for Lie Algebra Proofs

> Extracted from the image restriction proof audit (2026-07-09).
> These are non-obvious techniques for isolating coefficients in
> half-Leibniz bracket expansions.

## 1. Project to (u, v+1) Instead of (u, v)

**Context**: You have a commuting pair (E_{i,i+1}, E_{v,v+1}) and want to
isolate π_{uv}(φ(E_{i,i+1})) from the half-Leibniz equation. Projecting to
(u, v) often gives 0=0 because both brackets vanish at that target.

**Fix**: Project to (u, v+1) instead. The bracket [E_{pq}, E_{v,v+1}] at
(u, v+1) picks up the term δ_{q,v}E_{p,v+1} with p=u, q=v, which
directly contributes π_{uv}(φ(E_{i,i+1})).

**Why it works**: The bracket formula [E_{pq}, E_{v,v+1}] produces
E_{p,v+1} when q=v. So the target column shifts from v to v+1, and the
desired coefficient at column v appears naturally.

**When to use**: 
- u > i (target row above source row)
- v < n (so v+1 ≤ n and E_{v,v+1} exists)
- The partner E_{v,v+1} commutes with the source E_{i,i+1}

**Example**: Source (1,2), target (3,4), n≥5.
- Project to (3,5): [φ(E_{12}), E_{45}] at (3,5) gives π_{34}(φ(E_{12})) ✓
- Project to (3,4): [φ(E_{12}), E_{45}] at (3,4) gives 0 ✗

## 2. Chain Bridge + Centeredness for Diagonal Elimination

**Context**: You have an equation involving π_{a,b}(φ(E_{c,d})) where the
source and target share indices, creating a "near-diagonal" entry.

**Fix**: Apply the chain bridge at the wider source that includes the
near-diagonal, using Corollary cor:diag (all diagonal entries are zero
for centered φ) to eliminate terms.

**Example**: π_{2,v+1}(φ(E_{v,v+1})) appears after a commuting-partner
identity. Chain bridge at source (2,v+1) with split v gives:
2π_{2,v+1}(φ(E_{2,v+1})) = π_{2v}(φ(E_{2v})) + π_{v,v+1}(φ(E_{v,v+1})).
Both right-hand terms are diagonal entries → zero.

## 3. Commuting Partner Selection Rules

For source E_{i,i+1} and target (u,v)∉I:

| Condition | Partner | Commutes? | Target to project | What it isolates |
|-----------|---------|-----------|-------------------|-----------------|
| u=i, v≥i+2 | E_{v,v+1} | Yes (sets disjoint) | (i, v+1) | π_{iv}(φ(E_{i,i+1})) |
| u>i | E_{v,v+1} | Yes (v≥u+1≥i+2) | (u, v+1) | π_{uv}(φ(E_{i,i+1})) |
| u<i, v≠i+1, u>1 | E_{1,u} | Yes (1<u<i) | (1, v) | π_{uv}(φ(E_{i,i+1})) |
| u<i, v=i+1 | Two partners needed | — | 2×2 system | π_{u,i+1}(φ(E_{i,i+1})) |

## 4. The 2×2 System Pattern

When a single commuting partner cannot isolate the coefficient (because
the partner overlaps with source indices), use two commuting partners to
create a 2×2 linear system.

**Required**: Both partners must commute with the source, and the two
equations must be linearly independent.

**Pitfall**: The paper must either solve the system explicitly or state
that the unique solution forces the coefficient to zero. Writing "A 2×2
linear system forces..." without showing it is a gap — fill it in the appendix.

## 5. Induction on Column Index

For targets with v < n (Case 1 of image restriction), a column-induction
argument can unify many subcases:

- Base: v = n-1 (rightmost non-n column). Prove directly.
- Step: assume all targets with column > v are zero, prove for column v.

This reduces the number of commuting-partner choices needed.
