# Validate Theorem Statements Computationally Before Proving

## The rule

**Always validate classification-theorem statements computationally before
investing in Lean proof architecture.** This single rule prevented months of
wasted work (2026-06-22: caught dim=n → dim=n+5 falsification of HalfDer(N_n)).

## Methodology (30 minutes vs months of proof architecture)

### Step 1: Build full constraint matrix

For each pair of basis elements (E_ij, E_kl) and each target (E_uv), write the
half-Leibniz condition as a linear equation in the unknown coefficients
c(i,j; u,v). Gather all equations into a matrix A over ℚ.

### Step 2: Compute nullspace via Gaussian elimination

Use exact rational arithmetic (Fraction). Compute rank and nullity.

### Step 3: Cross-verify with Lean

Extract the first few nullspace vectors, construct them as coefficient
functions, and feed them into `bracketSource`/`bracketIdentity` via
`lean_run_code`. Zero violations confirms the nullspace is correct.

### Step 4: Extract structural patterns

Classify nullspace basis vectors by source, target column, coupling structure.
Look for ideals (subspaces closed under bracket), cocycles (coupled entries),
and n-independent patterns.

### Step 5: Paper-audit the simplest counterexample

Verify the simplest non-trivial nullspace vector satisfies ALL half-Leibniz
constraints by hand. If it passes, the original theorem is falsified.

## The N_5 counterexample that broke dim=n

```python
T(E_12) = E_25, all other T(E_ij) = 0
```

Verified: 0 violations against Lean's bracketSource/bracketIdentity.
This single vector added 1 dimension to the centered nullspace, proving
dim ≥ n+5 instead of the claimed n.

## Key insight: column-n entries are transparent

E_{*,n} entries bracket to zero with almost all E_kl because:
[E_{a,n}, E_kl] = δ_{n,k}E_{a,l} - δ_{l,a}E_{k,n}
δ_{n,k} = 0 (k < n always in N_n)
δ_{l,a} = 0 unless l = a (rare)

This makes column-n entries "invisible" to the half-Leibniz system,
allowing extra degrees of freedom that traditional bracket-chasing proofs miss.
