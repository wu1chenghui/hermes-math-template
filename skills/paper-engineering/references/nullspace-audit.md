# Nullspace Audit Technique & N_n Counterexample

## The technique: Linear-system verification of formal proofs

When a formal proof claims a classification (e.g., "all solutions have property P"),
verify by:

1. **Set up the full linear system** for small n (e.g., n=5) using the defining axioms
   (half-Leibniz, centering) — in Python with `Fraction` arithmetic for exactness.
2. **Compute the nullspace** via Gaussian elimination. Dimension > claimed → counterexample exists.
3. **Extract nullspace basis vectors** and interpret them as candidate derivations.
4. **Cross-validate** with the project's own Lean axiom checker (`bracketSource`/`bracketIdentity`)
   via `lean_run_code`.
5. Only proceed with Proof Reconstruction when nullspace dimension matches the claim.

This caught a genuine mathematical error **before** Lean code was written to "prove" it.

## N_5 Counterexample

**Claim** (the paper): All centered 1/2-derivations on N_n (n≥5) have offdiag=0.
Only nonzero component is the center (1,n) for adjacent sources.

**Found**: T(E_12) = E_25 is a valid centered 1/2-derivation on N_5.
- Nullspace over ℚ has dimension 6, not 1.
- Cross-validated by Lean's own `bracketSource`/`bracketIdentity`: 0 violations.

**Why it works**: Column-n entries (E_*,n) are transparent to most Lie brackets:
- [E_ab, E_kl] has δ_{b,k} term — b=n → k=n → no valid source (k<n)
- [E_kl, E_ab] has δ_{l,a} term — only nonzero when l=a, e.g., [E_12, E_25]=E_15

For T(E_12)=E_25:
- [E_12, E_25] = E_15: LHS=2·T(E_15)=0, RHS=[E_25,E_25]+0=0 ✓
- All other brackets: E_25 commutes or gives matched cancellation

**Nullspace structure (N=5):**
```
Vec 0: T(E_12)[1,5]=1           (center at endpoint 2)
Vec 1: T(E_12)[2,5]=1           ← counterexample (non-center offdiag!)
Vec 2: T(E_23)[1,5]=1           (center at endpoint 3)
Vec 3: T(E_13)[1,5]=1/2, T(E_23)[2,5]=1
Vec 4: T(E_34)[1,5]=1           (center at endpoint 4)
Vec 5: T(E_45)[1,5]=1           (center at endpoint 5)
```

Observation: source (1,2) has TWO independent degrees of freedom.

**Implication**: The classification theorem must be revised to either:
- Exclude column-n offdiag entries explicitly, or
- Redefine "centered" to eliminate them, or
- Accept the higher-dimensional nullspace as correct

## Bracket-family analysis (SCC-breaking technique)

**The problem**: Analyzing a SINGLE bracket (e.g., (n-2,n-1)) gave apparent SCC:
A₂ ↔ B (width-2 and adjacent source at endpoint j form mutual dependency).

**The solution**: Analyze the FULL family {(k, n-1) : k=1,…,n-2}. The system becomes:

```
V_k = 0  for k ≤ n-4  (directly, via two independent bracket expressions)
V_{n-3} = 0            (via bracket at (n-3,n-1) + (n-3,n-2))
V_{n-2}: 3t = 0        (via bracket at (n-2,n-1) + (n-2,n))
```

This transforms the apparent SCC into a DAG (unidirectional dependency).

**Pattern**: When individual equations appear cyclic, consider the FULL family of equations
indexed by a natural parameter. The matrix of the full system may be triangular even when
individual submatrices aren't.

**Python implementation pattern** (see references/nullspace-check.py):
- Build linear system from Lie bracket algebra
- Use Fraction for exact rational arithmetic
- Gaussian elimination → rank/nullity
- Cross-validate with Lean's bracket checker
