# Constraint Provenance Audit

> **When to use**: After the paper draft is stable and all lemmas are
> proven, but BEFORE finalizing the main theorem proof sketch. This audit
> asks: "Do we actually know WHICH constraints eliminate WHICH degrees
> of freedom, or are we just hand-waving the counting?"

## The Problem

A common failure mode in classification proofs: the paper claims
"Constraint Type X eliminates (n-1) degrees of freedom" but when you
actually compute the constraint matrix, Type X is vacuous for interior
cases and only contributes 1-2 constraints at the endpoints. The theorem
is still true, but the EXPLANATION is wrong. This creates a ticking
time bomb for referees.

## The Method

### Step 1: Fix a small n (e.g., n=5)

Don't try to verify the counting argument for general n. Pick a concrete
small n large enough that the general structure is visible but small
enough for full computation.

### Step 2: Build the FULL constraint matrix over Q

Use exact rational arithmetic (Python `fractions.Fraction`). Include ALL
variables. Don't use the paper's reduced variable set — the reduced set
may already encode incorrect assumptions about which constraints matter.

### Step 3: Compute rank layer by layer

Add constraints one mechanism at a time (HL only, HL+Image, HL+Image+Chain, etc.).
Record the marginal rank contribution of each layer. Identify which layer
contributes the most rank — that's the DOMINANT constraint.

### Step 4: Test specific claims

For each constraint type the paper claims eliminates DOFs:
- Write the explicit linear equations for n=5
- Check whether they are actually non-trivial (not 0=0)
- Count how many independent constraints they provide
- Compare with the paper's claimed count

### Step 5: Reconcile with Lean

Check what mechanism Lean actually uses. Often Lean encodes the dominant
constraint as a DEFINITION (e.g., `kerPhi = {f | f(i,j,u,v)=0 when (u,v)∉I}`)
rather than deriving it from HL equations. This tells you the REAL mechanism.

### Step 6: Rebuild the counting argument

Based on computational evidence, restructure the paper's upper bound proof
to reflect the ACTUAL constraint provenance. The corrected argument is
often simpler than the original — the dominant constraint becomes a
definition or direct structural consequence, not a complicated lemma.

## Case Study: Self-Pair Vacuity in N_n

For the 1/2-derivation classification on N_n, the original paper claimed
"self-pair constraints eliminate (n-1) degrees of freedom." For n=5:

- Built the full 100x100 constraint matrix
- HL only: rank 40 (60 free DOFs)
- HL + Image restriction: rank 90 (10 free DOFs = n+5)
- Image restriction contributed 50 rank vs 40 from HL → DOMINANT

Then tested the self-pair claim directly:
- For centered T0 at interior source (k,k+1), [T0(E_k), E_k] + [E_k, T0(E_k)] = 0
- E_{1,n} is central → zero contribution
- E_{1,n-1} only brackets non-trivially at k=n-1
- E_{2,n} only brackets non-trivially at k=1
- At k=1: BL term = -c1[E_{2,n},E_{12}] = +c1·E_{1n}, BR term = -c1[E_{12},E_{2,n}] = -c1·E_{1n}
  → BL + BR = 0 (CANCELS). At k=n-1: [E_{1,n-1},E_{n-1,n}] = E_{1n} and
  [E_{n-1,n},E_{1,n-1}] = -E_{1n} also cancel.
- Result: 0=0 for ALL adjacent sources, including endpoints. Self-pair is COMPLETELY VACUOUS.

**Conclusion**: Self-pair contributes ZERO constraints — not n-1, not even 1.
c_1 is NOT forced to zero; it survives as a free boundary parameter (basis B).
The paper §5.5 was corrected to say \"self-pair is vacuous for every adjacent source.\"

## The "Don't Change the Theorem, Verify First" Principle

When a proof technique fails for a boundary case (e.g., n=4 degenerates
the T1/T3 classification), do NOT immediately change the theorem
statement. Instead:

1. Compute the actual dimension/number/result for that boundary case
2. If the theorem still holds → the proof needs adjustment, not the theorem
3. If the theorem genuinely fails → THEN change the statement and add a remark

For N_4: the T3 vacuity analysis broke, but direct computation showed
dim(HalfDer(N_4)) = 10 ≠ n+5. The theorem genuinely needed n>=5, not n>=4.
This was discovered by computing the actual dimension, not by assuming
the proof failure meant the theorem failed.
