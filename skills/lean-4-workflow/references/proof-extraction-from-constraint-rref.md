# Proof Extraction from Constraint Matrix RREF

> When a coefficient is computationally verified to be forced to zero by the
> constraint system, but no single local equation constrains it directly, the
> proof is a linear combination of equations — a **syzygy**. This document
> describes how to extract human-readable proofs from the RREF of the full
> constraint matrix.

---

## When to use

- A sorrie is classified as MATH (true but unproven, not a Lean engineering gap)
- Experimental nullspace confirms the coefficient is zero in all solutions (World A)
- All local audits (occurrence, partner, interface) are exhausted
- You need to find WHICH equations combine to force the coefficient to zero

## The methodology

### Phase 1: Build the full constraint matrix

Build every half-Leibniz equation as a linear equation in the coefficient
variables. Include centered/diagonal conditions. Do NOT include `I_filtered`
or any hypothesis you're trying to prove — use only the defining axioms.

### Phase 2: Compute RREF with source tracking

Row-reduce the constraint matrix. Track, for each RREF row, which original
equations contributed to it. This is done by augmenting each row with an
identity vector representing the linear combination of original equations.

### Phase 3: Extract the syzygy for the target coefficient

If the target coefficient is a pivot column, its RREF row gives the exact
linear combination of original equations that isolates it:

    target_coeff = Σ c_i · eq_i

Since each eq_i = 0 (by the defining axiom), target_coeff = 0.

### Phase 4: Interpret the syzygy

The linear combination may involve dozens of equations. Simplify:
- Some equations are redundant (eliminated in RREF)
- The RREF stores a compressed form; back-substitute to find the minimal set
- The minimal set is the **proof** — each equation = a half-Leibniz identity
- The combination = a polynomial identity (cancellation of intermediate terms)

## The 3-equation syzygy pattern (Boundary Rigidity case study)

For `coeff(i,i+1; i,n)` with i ≥ 3:

Three half-Leibniz equations form a 3-variable system in {a, b, g}:
```
a = coeff(1,i+1; 1,n)    [wide-source b-channel]
b = coeff(2,i+1; 2,n)    [wide-source b-channel]
g = coeff(i,i+1; i,n)    [GOAL]

Eq_A: src=(1,i), ptr=(i,i+1), tgt=(1,n)  →  2a - g = 0
Eq_B: src=(1,2), ptr=(2,i+1), tgt=(1,n)  →  2a - b = 0
Eq_C: src=(2,i), ptr=(i,i+1), tgt=(2,n)  →  2b - g = 0
```

Syzygy: (-2)·Eq_A + 2·Eq_B + Eq_C = g

The a and b terms cancel algebraically, isolating g. Since each Eq = 0
(half-Leibniz axiom), g = 0. No `I_filtered`, no width stability, no induction.

## Significance

This methodology converts a "mysterious global constraint" into an explicit
linear combination of known equations. The result is a concrete proof that
can be directly formalized in Lean.

## Pitfalls

- Do NOT trust the RREF equation count alone. Some pivots are established by
  equations that themselves need other pivots. Trace the dependency tree.
- The "2 equation" output from source-set tracking can be misleading: the RREF
  already incorporates earlier pivots. The raw linear combination (from
  augmented matrix tracking) shows all truly participating equations.
- For the `compute_nullspace_dim.py` script, use exact rational arithmetic
  (Python `fractions.Fraction`), not floating-point. The syzygy coefficients
  are small integers.
