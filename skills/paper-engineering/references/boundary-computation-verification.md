# Boundary Computation Verification & Proof-Sketch Audit

> Two tightly related techniques discovered during the N_n paper audit
> (2026-07-02). Both involve **verifying paper claims with computation
> before trusting the prose.**

---

## Technique 1: Small-n Boundary Computation

### When to use

- The paper's proof uses an induction or propagation argument that
  requires a minimum n to close (e.g., "the induction step works for
  n≥5").
- The theorem statement says n≥4, but the proof sketch implicitly
  assumes n≥5 at some point.
- A structural classification (e.g., T₀–T₃ generators) changes
  character at small n because indices that are distinct for large n
  coincide for small n.

### Method

1. **Identify the suspicious claim.** Look for conditions like
   "for n≥5", "interior k", "k not at the endpoints", or index
   comparisons like "j ≤ n−2" that fail at small n.

2. **Build the full constraint matrix for the smallest n in question**
   (usually n=4). Use EXACT rational arithmetic (Python `fractions`
   module, not floating-point). Include ALL unknowns — every
   source→target pair, not just the "interesting" ones.

3. **Compute the nullspace dimension.** Compare with the paper's
   predicted dimension (n+5). If they differ, the theorem statement
   needs adjustment.

4. **If the dimension differs, find the extra basis vector.**
   Compute the nullspace explicitly. Project it orthogonal to the
   paper's claimed basis. The residual reveals the extra degree of
   freedom.

5. **Derive the analytic form.** For the extra vector, solve the
   constraint equations analytically (e.g., pure diagonal derivation
   equations). This confirms the computational finding is not a
   numerical artifact.

6. **Adjust the theorem.** Three options:
   - (a) Change n≥4 to n≥5 (least invasive, most common).
   - (b) Add a separate small-n computation as a remark.
   - (c) Rewrite the proof to cover the small-n case.

### The N_n case study

- Theorem claimed n≥4, dim = n+5.
- T₃ vacuity analysis in §5.5 used condition "3 ≤ n−2 for n≥5".
- For n=4: (1,3) and (3,4) share index 3 → [E₁₃,E₃₄]=E₁₄≠0 → they
  are T₁ generators, not T₃. The T₃ vacuity analysis does not apply.
- Exact rational computation: dim(HalfDer(N₄)) = 10 = n+6, not n+5=9.
- Root cause: diagonal propagation chain too short — additional
  equation [E₁₂,E₂₄]=E₁₄ creates a coupling absent for n≥5.
- Resolution: change theorem to n≥5, add Remark explaining n=4
  exceptional case with dim=10.

### Key principle

**Do not change the theorem based on proof-technique failure.**
First verify the theorem itself (via computation). Only change the
theorem if the computation shows it's actually false for that case.
If the theorem holds but the proof needs adjustment, that's a different
category of fix.

---

## Technique 2: Proof-Sketch Claim Verification

### When to use

- A paper proof makes an explicit numeric claim like "eliminates n−1
  degrees of freedom", "gives one linear relation per source",
  "forces all X to zero".
- The claim involves a counting argument that wasn't verified in Lean.

### Method

1. **Extract the claimed relation.** Write down exactly what equation
   the paper says follows from the half-Leibniz condition.

2. **Compute it for concrete small parameters.** Pick n=5 (or the
   smallest n where the proof applies). For EACH value of the relevant
   index, expand the bracket and check whether the claimed relation
   actually holds.

3. **Classify the result per index.** Some indices may give non-trivial
   constraints; others may give 0=0 (vacuous). The paper's claim may
   only be true for endpoint indices.

4. **If the claim fails, identify the CORRECT constraints.** Trace
   which other mechanisms (cross-source chain, F_eq bridge, channel
   restrictions) actually provide the missing constraints.

### The self-pair case study

- §7 of the N_n paper claimed: "Self-pair constraints yield one linear
  relation per adjacent source, eliminating n−1 degrees of freedom."
- Computational verification for n=5: ALL self-pair brackets are
  identically 0=0 for interior sources (2 ≤ k ≤ n−2). Only at the
  Dynkin endpoints (k=1 giving c₁=0, k=n−1 vacuous) does any
  non-trivial constraint emerge.
- Root cause: T₀(E_{k,k+1}) = a_k·E_{1,n-1} + b_k·E_{1,n} + c_k·E_{2,n}.
  E_{1,n} is central (brackets to 0 with everything). E_{1,n-1} only
  brackets with E_{n-1,n}. E_{2,n} only brackets with E_{12}. For
  interior k, neither endpoint source appears.
- The constraints that ACTUALLY eliminate degrees of freedom come from
  cross-source chain equations and channel restrictions, not self-pair.

### Key principle

**Never trust a counting claim in a paper proof sketch without
verifying it computationally for at least one concrete parameter set.**
The prose may describe what the author THINKS happens, not what the
equations actually produce.
