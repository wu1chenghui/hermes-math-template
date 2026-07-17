# coeffOf_cond_zero: Bracket expansion bridge

## What it is

The `coeffOf_cond_zero` lemma bridges the `HalfDerivation.cond_zero` property
from the HalfDerivation world (MatIdx-indexed, `dite`) to the NCoeff world
(ℕ-indexed, `ite`). It encodes the identity:

```
[D(E(i,j)), E(c,d)] + [E(i,j), D(E(c,d))] = 0
```

when `[E(i,j), E(c,d)] = 0` (i.e., `j ≠ c` and `d ≠ i`).

## Status

🟡 **Frozen** as of June 2026. The proof requires bracket expansion in the
coefficient basis — a mechanical but lengthy combinatorial argument.
It is NOT needed for the main classification chain because the Adjacent
analysis can be done directly via `half_deriv_cond` + `coeffOf_cond` (which IS done).

## Why it's hard

The bracket `[D(E(i,j)), E(c,d)]` expands as:
- Σ_{p,q} coeffOf D i j p q · [E(p,q), E(c,d)]

For each (p,q), `[E(p,q), E(c,d)]` is:
- E(p,d) if q = c (with condition p < d)
- -E(c,q) if d = p (with condition c < q)
- 0 otherwise

The coefficient of E(u,v) in this sum comes from exactly TWO terms:
- p=u, q=c when v=d: coeffOf D i j u c
- p=d, q=v when u=c: -coeffOf D i j d v

Similarly for `[E(i,j), D(E(c,d))]`, the coefficient at E(u,v) comes from:
- q=v, p=j when u=i: coeffOf D c d j v
- p=u, q=i when v=j: -coeffOf D c d u i

Summing all four and equating to 0 gives the `cond_zero` equation.

## Where it's needed

- `NCoeff.corner_relation` (Adjacent.lean:278) — uses `D.cond_zero`
- `NCoeff.adjacent_reduce_to_corner` — uses lemmas that use `cond_zero`
- `ImageInCenter.T_support_adjacent` — calls `adjacent_reduce_to_corner`
- `ImageInCenter.coupled_relation` — calls `corner_relation`

All of these compile because `toNCoeff` has `cond_zero := coeffOf_cond_zero D`
which is `sorry` (accepted as an axiom by Lean).

## Planned resolution

### 方案 C (recommended, June 2026)

See `references/coeffOf-cond-zero-strategy-C.md` for the full strategy.

The proof should establish the 4-term identity as a **bracket expansion identity**
via one of two paths:

**Path 1 (bracket expansion bridge):** Build `HalfDerivation.toLinearMap`,
define bracket on `𝔫`, prove `[D(E_ij), E_cd]` expands term-by-term.
~150-250 lines of infrastructure, then `coeffOf_cond_zero` is ~10 lines.

**Path 2 (coeffOf_cond bidirectional):** Use `coeffOf_cond` to relate surviving
terms across the two sources without building the full linear-map infrastructure.

Key principle: `Bracket.lean` must remain a **local combinatorial identity library**
that depends only on `Infrastructure.lean`. It must NOT call `coeffOf_nonadjacent`,
which creates a cycle with the classification modules.
