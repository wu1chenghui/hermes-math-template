# Experimental Nullspace Verification — Compute Before Proving

## When to use

When a blockage has been classified as MATH (genuine gap, not engineering) by the
blocker-verification audit, and you need to determine whether the statement is
actually TRUE before investing in a proof.

This is "World A vs World B" disambiguation:
- World A: statement is true → need new proof
- World B: statement is false → classification theorem needs revision

## The method

1. Build the FULL linear constraint system (all half-Leibniz equations) as a
   sparse matrix over ℚ (exact rational arithmetic, Fraction).
2. Add auxiliary conditions (e.g., centered = all diagonal coefficients zero).
3. Do NOT add the condition you're testing (e.g., I_filtered).
4. Gaussian-eliminate (sparse row reduction, pivot-based).
5. Extract nullspace = kernel; compute dimension.
6. Compare with known answer (e.g., dim kerΦ = n+4 from Spanning).

## Case study: Boundary Rigidity (2026-06-28)

**Question**: Is `dim(C) = n+4` where `C = {centered half-derivations}` (no I_filtered)?

**Setup**: n=5,6,7. Variables = all `coeff(i,j;u,v)` for valid (source,target) pairs.
Equations = all half-Leibniz `2·bracketSource - bracketIdentity = 0` +
centered condition `coeff(i,j;i,j) = 0`.

**Result**:
| n | variables | equations | rank | dim C | kerΦ | Verdict |
|---|-----------|-----------|------|-------|------|---------|
| 5 | 100 | 450 | 91 | 9 | 9 | C = kerΦ |
| 6 | 225 | 1395 | 215 | 10 | 10 | C = kerΦ |
| 7 | 441 | 3521 | 430 | 11 | 11 | C = kerΦ |

**Conclusion**: Boundary Rigidity is TRUE. `C = kerΦ`. There MUST exist a non-local
constraint that forces the v=n coefficient to zero, even though every individual
half-Leibniz equation involving it degenerates to 0=0 after centering.

## The global constraint phenomenon

The 450+ equations collectively constrain the system, but any SINGLE equation
involving `coeff(i,i+1;i,n)` is trivial. This is a "global syzygy" — a linear
dependence among constraint rows that becomes non-trivial only when ALL rows are
considered simultaneously.

The computational experiment confirms the theorem but does not reveal the proof.
The next step is to extract the specific linear combination of constraint rows
that forces the coefficient to zero, then translate it into a mathematical argument.

## Script

See `scripts/compute_nullspace_dim.py` for the reusable implementation.
Uses Python stdlib `Fraction` for exact rational arithmetic, sparse row reduction.
Works for n ≤ 7 (n=8 would need ~700 variables × ~7000 equations, may need
optimized sparse solver).
