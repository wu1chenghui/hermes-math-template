# SCC Extraction Before Proof Search

## When to use this reference

You have a target coefficient `coeff(i,j;u,v)` you need to prove is zero, and you've
found ONE bracketIdentity or coeffOf_cond relation involving it, but can't find a
second independent equation.  **Stop searching for partners.**  You need to extract
the full SCC first.

## The pattern

```
✓ Found: 2·B = A   (one equation, two unknowns)
✗ Searching: partner (c,d), target (u',v') — all give same equation or irrelevant terms
✗ Searching: different split points — same
✗ Searching: σ-mirror — introduces diagonal contamination
```

**Root cause**: The true SCC contains more than 2 variables.  The single equation is
just one edge in a larger closure.

## Method: full dependency graph extraction

1. **Start variable**: `X₀ = coeff(i,j;u,v)`

2. **For each variable `Xₖ = coeff(a,b;c,d)`**, enumerate ALL equations:
   - `diagonal_shift`: half-Leibniz with partner `(j, j+1)`, target `(c, j+1)`,
     gives `coeff(a,j;c,j) = 2·coeff(a,j+1;c,j+1)` [only T1 fires]
   - `coeffOf_cond` at split `m` (`a < m < b`): gives `2·Xₖ = A - B + C - D`
   - `bracketIdentity` with commuting partner `(p,q)` and target `(r,s)`:
     T1-T4 may fire, each introducing new variables

3. **Record every NEW coefficient** that appears.  Don't eliminate — just list edges.

4. **Stop when the graph closes** (no new variables appear) or when you hit
   variables of a DIFFERENT type (diagonals, boundary coefficients).

5. **Analyze the graph**:
   - If it forms a pure cycle on N variables with N independent equations: **local N×N SCC**
   - If the graph leaks into diagonals: **induction predicate is wrong** — the
     natural invariant should include those variables
   - If the graph leaks into a boundary cluster: **this is a genuine interface**
     between D3 and another layer

## Red flags

### Characteristic-dependent chains
If the equations chain like `A = 2·B = 4·C = 8·D = ...`, producing powers of 2,
**stop immediately**.  This leads to `(2^m - 2)·x = 0` which is characteristic-
dependent.  This is the wrong mechanism — the true SCC has a constant determinant.

### Single equation, no second
If ALL bracketIdentity + coeffOf_cond variations produce the SAME equation (just
`X = 2·W` or `X = 2·Y`), the SCC has NOT been fully extracted.  The current
induction predicate may be projecting away essential variables.

## Example: A-fire 3×3 SCC

```
Target: coeff(k,k+1;1,k+1)  [A-fire child]

Graph:
  A = coeff(k,k+1;1,k+1)
  │  diagonal_shift at k+1
  ▼
  B = coeff(k,k+2;1,k+2)
  │  diagonal_shift at k+2
  ▼
  C = coeff(k,k+3;1,k+3)
  │  coeffOf_cond(k,k+3;1,k+3) at k+1
  └──► A = 2·C

Equations: A = 2B, B = 2C, A = 2C  →  3×3, det=2, closed.
```

## Example: C-fire (failed SCC)

```
Target: coeff(3,j';3,n)  [C-fire child, k=2]

Graph leaks to:
  coeff(2,j';2,n)  [width_c goal, proportional via 2·W = X]
  coeff(1,j';1,n)  [further propagation]
  coeff(1,2;1,2)   [DIAGONAL — different type!]

Conclusion: induction predicate P(ℓ) = coeff(ℓ,j';2,n) is too weak.
The natural invariant is Q(ℓ) = ∀t≤ℓ, coeff(t,j';t,n)=0.
```

## Anti-patterns to avoid

- **Blind partner search in Lean**:  Writing `bracket_zero` calls with every
  conceivable partner, trying to compile.  This is the DFS-on-graph approach
  done manually and slowly.
- **Exponential chain acceptance**:  Taking `A = 2^{m}·C` as a valid proof
  strategy.  The characteristic-dependence means it's not a proof for all
  fields.
- **Premature lemma creation**:  Writing a `c_fire_chain_close` lemma before
  verifying the terminal can be isolated.  The chain just moves the problem.
