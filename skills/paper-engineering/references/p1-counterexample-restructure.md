# Theorem 6.3 Restructure — P_1 Counterexample Finding (June 2026)

## The Original (Incorrect) Statement

> Theorem 6.3: For every source I and every target T ≠ I, c_I^T = 0.

This is **mathematically false**. Counterexample: P_1(E_{12}) = E_{1n} is a valid
1/2-derivation with c_{12}^{1n} = 1 ≠ 0. The identity map and central perturbations
together give dim = n, not dim = 1.

## The Corrected Structure

Split into two theorems:

**Theorem 6.3 (Nonadjacent Sources)**: For w(I) ≥ 2 and T ≠ I, c_I^T = 0.

**Theorem 6.3b (Adjacent Sources)**: For I = (i,i+1) and T ≠ I, T ≠ (1,n), c_I^T = 0.
The remaining degree of freedom (coeff at (1,n)) gives the n-1 central parameters.

## Why the Induction is NOT Contaminated

The induction in coeffOf_nonadjacent creates width-1 child sources, but the child
targets are NEVER (1,n). The four recursive branches all satisfy:
- IA: child target second coord = i+1 < n
- IB: child target first coord = j > 1
- IIA: child target first coord = i+1 ≥ 2
- IIB: child target second coord = i < n

See: lean-4-proof-writing/references/induction-contamination-check.md

## Paper Restructure

```
Old (§6):
  Thm 6.3: all offdiag = 0 (wrong)
  Thm 6.4: D = λId + T with T(N_n) ⊆ Z(N_n)

New (§6):
  Thm 6.3: nonadjacent sources → all offdiag = 0
  Lemma 6.x: adjacent sources → offdiag = 0 except (1,n)
  Thm 6.4: D = λId + Σ a_i P_i (classification)
  Corollary: dim = n
```
