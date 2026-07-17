# Paper ↔ Lean Proof-Level Consistency Audit

> Aligns paper proofs with Lean proofs at the proof-chain level, not just
> theorem-name level. Developed during the 1/2-derivation classification
> project (2026-07-03).

## When to run

After the paper's proof sketch is stable and the main Lean theorems are
proved. This is a pre-submission verification, not a research tool.

## Method

### Step 1: Extract paper claims

List every Lemma, Theorem, and Corollary in the paper with its
mathematical statement and section location.

### Step 2: Find Lean counterparts (proof-level, NOT name-level)

For each paper claim:

1. Use `lean_file_outline` on the relevant Lean module to list all declarations
2. Compare TYPE SIGNATURES, not just names
3. Read the Lean proof body to understand what it actually concludes
4. Verify the Lean conclusion matches the paper claim

**Critical anti-pattern**: matching `boundary_rigidity` to "F_eq bridge"
because they sound related. In the N_n project, `boundary_rigidity` proves
`coeff(i,i+1;i,n)=0` (interior v=n coefficient), while F_eq is `a_1+c_{n-1}=0`
(endpoint coupling). The real Lean counterpart to F_eq is `F1_coeff_relation`
(p=1 branch) + `F2_coeff_relation` (p=n-1 branch).

### Step 3: Trace proof chains

For each major paper result, reconstruct the Lean proof chain:

```
Paper step → Lean lemma → Lean sub-lemma → conclusion
```

Example for F_eq:
```
Paper: a_1 + c_{n-1} = 0
Lean: F1_coeff_relation (p=1 branch)
  ↓
  commuting_adj_F1 → bracketSource_commuting_eq_zero → Phi=0
  ↓
  bracketIdentity_at_1n_canonical → indicator expansion
  ↓
  a_1 + c_{n-1} = 0
```

### Step 4: Verify dependency DAG alignment

Check that the paper's dependency order matches Lean's dependency order.
No reverse dependencies, no missing steps.

### Step 5: Check for "stronger in paper than Lean"

Verify no paper claim exceeds what Lean actually proves. If the paper says
"all centered derivations satisfy X" but Lean only proves it for "all
finite-support centered derivations," flag it.

## Common pitfalls

- **Theorem-name matching**: `boundary_rigidity` ≠ F_eq. Always read the
  type signature, not just the name.
- **Missing Lean lemmas**: Paper may decompose a result into multiple lemmas
  while Lean proves it in one (width reduction is embedded in Centering, not
  a standalone lemma). This is fine — different proof granularity.
- **Dead code**: Lean may contain legacy theorems (`offdiag_zero`,
  `classification`) from old proof attempts. Check that paper doesn't
  accidentally claim what those dead theorems state.
