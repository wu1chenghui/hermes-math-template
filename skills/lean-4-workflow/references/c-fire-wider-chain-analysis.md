# C-Fire Wider Chain: Dependency Probe Methodology

**When**: a `coeffOf_cond` expansion produces a child coefficient that is not
covered by the current induction hypothesis, and the child belongs to a different
proof domain (e.g., c-channel chain produces an off-diagonal ∉I child).

**Why**: before changing induction predicates or restructing lemmas, determine
whether the child is locally closeable or creates a structural cycle.

## The probe

1. **Pick a concrete instance** — e.g., `coeff(3,6;3,8)` for n=8.
2. **Trace `coeffOf_cond` at all valid split points** — list the children.
3. **Trace `bracketIdentity`** for each child — identify coupling to other
   coefficient families (b-channel, diagonal, etc.).
4. **Draw the full dependency graph** — follow each child until it either
   terminates (zero-residual, self-contained equation) or cycles back to a
   parent (structural cycle).
5. **Check centering sensitivity** — do diagonal intermediates appear? Are they
   needed for closure?

## Worked example (this project, wider C-fire)

For `coeff(3,j';3,n)` with j'≥5, j'<n:

```
coeff(3,j';3,n)
  │ coeffOf_cond at k=4
  ├── coeff(3,4;3,4)      ← DIAGONAL adjacent
  └── coeff(4,j';4,n)
        │ coeffOf_cond at k=5
        ├── coeff(4,5;4,5)  ← DIAGONAL
        └── coeff(5,j';5,n)
              │ ...
              └── V(j'-1)
                    │ half-Leibniz T2
                    └── coeff(1,j';1,n)  ← b-channel
                          │ width_b_zero
                          └── coeff(2,j';2,n)  ← PARENT ← CYCLE
```

**Verdict**: NOT locally closeable. The chain introduces diagonal intermediates
at every step and eventually cycles through b-channel → width_b_zero → parent.
Width-2 (k=2, j'=k+2=4) is the ONLY case where this chain collapses to length 1
(adjacent source has no intermediate diagonal child).

**With centering** (all diagonal = 0): the chain simplifies to `2^{j'-3}·X =
2·coeff(1,j';1,n)`. But `coeff(1,j';1,n)` is b-channel nonadjacent — NOT forced
to 0 by centering alone. The closure still needs width-collapse machinery,
which reintroduces the cycle.

## Why width-2 worked (and wider doesn't)

For `coeffOf(2,4;2,n)` at split k=3: child = `coeffOf(3,4;3,n)` = V(3). This
is ADJACENT — no further split → no diagonal intermediate → three-equation
local 2×2 closure is possible:

```
coeffOf_cond(2,4;2,n) → V(3)
V(3) → half-Leibniz → coeff(1,4;1,n)
coeffOf_cond(1,4;1,n) → coeff(2,4;2,n)
→ 2x = x → x = 0
```

For wider sources (j'≥5), the child `coeff(3,j';3,n)` is NONADJACENT, forcing
a chain of coeffOf_cond splits, each introducing a diagonal intermediate.

## The blueprint's approach

The blueprint avoids this entire problem by using `width_stability_c` with
`hI : I_filtered`. Under hI, the child `coeff(3,j';3,n)` is trivially zero
because `(3,n) ∉ I`. No chain, no cycle, no diagonal intermediates matter.

The Lean implementation created `width_c_chain_zero` as an hI-free version,
which surfaces the structural cycle that hI was hiding.

## Decision heuristic

When a coeffOf_cond child is NOT covered by the current IH:
1. If the child is **adjacent** → local 2×2/3-equation closure possible
2. If the child is **nonadjacent** and chains through diagonal intermediates →
   NOT locally closeable without centering → belongs in a different proof layer
3. If the child is **trivially zero under a known hypothesis** (like hI) →
   check whether that hypothesis can be legitimately introduced at the call site
