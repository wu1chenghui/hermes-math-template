# Paper-Lean Proof Reconstruction Methodology

> Methodology developed during the N_n half-derivation classification project.
> Applies to any project where a Lean formalization exists and a human-readable
> mathematical paper must be written.

## Core principle

**Lean is a referee, not a producer.** The mathematical proof must be independently
reconstructed from first principles. Lean only verifies that the reconstruction
matches the formal proof tree.

## Three-dimension audit

Before any writing, audit the paper against Lean along three independent axes:

| Dimension | Question | Severity if fail |
|-----------|----------|:--:|
| A. Mathematical Correctness | Is the proof mathematically valid? | 🔴 CRITICAL — proof is wrong, not just incomplete |
| B. Proof Completeness | Is every step shown, or are steps omitted / deferred to Lean? | 🔴 — paper is not self-contained |
| C. Lean Consistency | Does the paper's proof tree match Lean's? | 🟡 — correct but different proof; harder to maintain |

A-dimension errors (e.g., using a vacuous identity like [A,B]+[B,A]=0 as a constraint)
are more severe than B-dimension gaps (e.g., "Lean verified this, omitted here").

## Reconstruction pipeline

```
1. STATEMENT EXTRACTION
   For each lemma, extract the exact mathematical statement from Lean.
   Do NOT read the Lean proof — only the type signature.

2. PROOF TREE VERIFICATION
   Verify that the Lean proof tree is mathematically minimal.
   Check: can any helper lemmas be merged? Are all splits essential?
   Ask: does each split correspond to a genuinely different mathematical case?

3. PATTERN IDENTIFICATION
   Classify all helpers into a small set of proof patterns
   (e.g., single-equation, multi-equation syzygy, induction).
   Write a unified Observation for any pattern used ≥2 times.

4. HELPER RECONSTRUCTION
   For each helper lemma, independently reconstruct the mathematical proof.
   Use only the statement + first principles. Do NOT reference Lean.
   Start with the simplest pattern; each subsequent helper reuses the template.

5. LEAN AUDIT (per helper)
   After reconstruction, verify four questions:
   Q1: Same number of Phi=0 / HL equations?
   Q2: Same source/partner/target identities?
   Q3: Same linear relations?
   Q4: Any extra assumptions (centeredness, I_filtered, etc.) not in the paper proof?

   If all four match: the reconstruction and Lean describe the same proof.
   If not: the reconstruction is a different proof — not necessarily wrong,
   but the paper will have mixed proof strategies across lemmas.

6. ASSEMBLY
   Combine all helpers into the main lemma following the verified proof tree.
   Write the dispatch logic (case analysis connecting helpers to cases).

7. PAPER REWRITE
   Only now open the .tex file. The proof is already complete.
```

## Unified Observation pattern

When bracket/coupling computations recur across multiple lemmas, extract
a single Observation used by all. Example from this project:

> **Observation (Bracket-coefficient criterion).** For [E_{p,q}, E_{r,s}]
> = δ_{qr}E_{ps} − δ_{sp}E_{rq}, the coefficient of E_{u,v} can only arise
> via Type I (δ_{qr}=1, p=u, s=v) or Type II (δ_{sp}=1, r=u, q=v).

This eliminates repetitive case enumeration across all helpers.

## State tracking

Use a status table throughout reconstruction:

| Lemma | Pattern | Status | Notes |
|-------|:--:|:--:|------|
| boundary_rigidity | A | Verified | 3-eq syzygy, Lean audit passed |
| adjacent_* | B | Draft | — |

Status levels: Not Started → Draft (proof written) → Verified (Lean audit passed + bracket audit clean).

## Pitfalls

- **"Verified in Lean, omitted here"**: NEVER acceptable in a math paper. If Lean has a
  proof, the paper must contain the mathematical argument.
- **Using Lean as a crutch**: Reading Lean proofs and translating them line-by-line
  produces verbose, unnatural math. Reconstruct independently, then compare.
- **Confusing "rebuild" with "complete"**: If the paper proof is based on an incorrect
  argument (e.g., a vacuous identity), it must be REBUILT from scratch, not just
  "completed" by filling gaps.
- **Skipping proof tree verification**: Lean often splits proofs into more lemmas than
  mathematically necessary (for automation, reuse, etc.). Verify the tree is minimal
  before reconstructing each node separately.
