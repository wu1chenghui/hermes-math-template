# Computational Experimentation Methodology

> How to build the half-Leibniz linear system, compute nullspaces, and extract syzygies
> to guide mathematical discovery. Used successfully to find the Boundary Rigidity proof.

## When to use

- A coefficient resists all local/partner-based proof attempts
- Need to determine whether a statement is true before investing in Lean formalization
- Want to extract the minimal set of equations that force a specific coefficient to zero

## Step 1: Build the full linear system

For n, enumerate all variables:
```python
st_list = [(a,b,e,f) for a in range(1,n+1) for b in range(a+1,n+1)
           for e in range(1,n+1) for f in range(e+1,n+1)]
```

For each (source, partner, target) triple, generate the half-Leibniz equation:
`2·bracketSource - bracketIdentity = 0`

Add centered condition: all diagonal coefficients = 0.

## Step 2: Compute nullspace dimension

Use sparse row reduction with Fraction (exact rational arithmetic).
Sort rows by sparsity before elimination for efficiency.
Rank = number of pivots; nullity = variables - rank.

Compare nullity with known bounds (e.g., dim kerΦ = n+4) to determine
if the unconstrained space equals the known subspace.

## Step 3: Check specific coefficients

After computing the nullspace basis, check whether a target coefficient
is zero in all basis vectors. If yes, the theorem is computationally verified.

## Step 4: Extract the syzygy

To find WHICH equations force the coefficient to zero:
- Track linear combinations during row reduction (augment each row with
  a dict mapping source equation index → coefficient)
- After RREF, the row for the target coefficient gives the linear combination
- Simplify to find the minimal set of participating equations

## Step 5: Translate to mathematics

The syzygy reveals which half-Leibniz equations combine to force the result.
Translate each equation back to (source, partner, target) notation.
Expand T1-T4 mechanically to verify correctness.
The result is a human-readable proof.

## Pitfalls

- Sorting by sparsity can cause zero-pivot issues; validate rank against known results
- The `symmetric_difference` tracking in source sets may lose information with non-unit coefficients; track full linear combinations instead
- Edge cases (n = i+1) require diagonal handling; verify both branches
- The extracted syzygy may depend on previously-eliminated pivots; trace the full dependency tree
