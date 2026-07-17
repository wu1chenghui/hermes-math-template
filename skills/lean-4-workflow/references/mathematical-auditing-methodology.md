# Mathematical Auditing Methodology

> Distilled from the 2026-06-30 session auditing the dim=n+5 classification proof.
> These are reusable patterns for auditing formalized mathematical proofs,
> independently of any specific Lean project.

## Core Principle: Attack, Don't Explain

When auditing a completed proof, the highest-value activity is NOT
explaining why each step works — it's actively trying to CONSTRUCT
violations. Every failed counterexample attempt increases confidence
more than any explanatory audit.

## Audit Hierarchy (Increasing Mathematical Depth)

### Level 1: Engineering Audits
- Clean build verification
- Module/call-graph analysis
- Dead code identification
- These answer: "Does the code work?"

### Level 2: Proof Audits
- Proof-graph sorrie check
- Hidden assumption audit (every logical step backed by a theorem?)
- These answer: "Is the proof chain complete?"

### Level 3: Mathematical Trust Audits
- Model audit (do definitions match math?)
- Specification audit (does each theorem prove exactly what's needed?)
- Freedom audit (no missing degrees of freedom?)
- Coverage audit (every object classified exactly once?)
- These answer: "Is the mathematical content correct?"

### Level 4: Attack Audits (Highest Value)
- **Counterexample attack**: Try to construct derivations violating each
  core theorem. Find the exact half-Leibniz equation where it fails.
- **Constraint coverage**: Build equation incidence matrix. Every
  coefficient must have ≥1 constraining equation.
- **Constraint rank**: Multiple witnesses must be genuinely independent
  (different variable sets in the Jacobian).
- **Dependency graph connectivity**: The constraint network must be a
  single connected component. Disconnected subgraphs = hidden degrees
  of freedom.

## The dim=n Failure Archetype

The original dim=n error was NOT a lemma mistake or a proof gap. It was:
1. Several DETERMINATION equations were misclassified as ELIMINATION
   equations. (Determination: relates parameters to each other. Elimination:
   forces a parameter to zero.)
2. A non-local constraint (F_eq, bridging left and right Dynkin ends)
   was completely invisible to the local analysis.
3. The constraint graph was DISCONNECTED — the left and right boundary
   blocks were separate components, and the 5 missing parameters lived
   in the gaps.

## Mathematical Certificates (Not Text Summaries)

For each audit finding, produce a machine-checkable certificate:

1. **Equation Coverage Matrix**: Rows = coefficients, columns = equations.
   Each cell = ● if the equation constrains that coefficient. Verify:
   - No row has 0 ●'s (every coefficient constrained)
   - No column has 0 ●'s (every equation constrains something)

2. **Constraint Rank Certificate**: For each free parameter with multiple
   witnesses, prove the witnesses are independent by showing they involve
   different variable sets in the Jacobian.

3. **Constraint Dependency Graph**: Nodes = coefficient variables.
   Edges = shared equation. Verify single connected component.

## Post-Hoc Interpretation Warning

When a computational result (e.g., "5 boundary parameters") is later
"explained" by a structural fact (e.g., "Serre relations give 2+2+1"),
distinguish:
- **Post-hoc**: "We found 5, and noticed it decomposes as 2+2+1."
- **Structural**: "The Dynkin diagram automorphism σ forces the orbit
  decomposition {2,2,1}. Therefore 5 is structurally necessary."

Only the second is a genuine mathematical explanation. To establish it,
prove that the decomposition is UNIQUE under the natural symmetries —
not just one of several possible interpretations.

## Equation Classification

Every constraining equation must be classified as one of:
- **ELIMINATION**: Forces the coefficient to ZERO. (e.g., bracket analysis
  shows no other term can cancel the contribution.)
- **DETERMINATION**: Relates the coefficient to OTHER coefficients,
  expressing it in terms of free parameters.
- **COUPLING**: Links two coefficients (e.g., F_a + F_c = 0), reducing
  the degrees of freedom by 1 without forcing either to zero.

The dim=n error: 5 DETERMINATION equations were treated as ELIMINATION.
