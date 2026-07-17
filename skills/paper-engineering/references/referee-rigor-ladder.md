# The rigor-closure ladder (finalization-stage hardening)

How to drive a formalized result from "geometrically correct" to
"linear-algebra-closed / referee-acceptable." Distilled from an extended
hostile-referee review of an upper-bound argument (N_n ½-derivation
classification, dim ker(Φ) ≤ n+4). The *methodology* generalizes to any
structural classification / dimension theorem.

This complements Pass 2 (Referee Attack Audit) in SKILL.md: that audit checks a
finished draft; this ladder is the iterative tightening you apply WHILE closing
the last gap, before anything gets locked into the paper.

---

## 0. User's review style (embed this — it governs the whole class)

This user runs an **escalating tightening review**: each turn accepts the
direction but isolates one more "极小但必须显式" gap and refuses to finalize
until it is closed. Expect, in order: framing → necessity → generation →
rank-realization → non-circular closure. **Do NOT present a step as "closed"
until it is at the top of the ladder.** Pre-empt the review: before claiming
closure, self-interrogate "is this necessity or generation? is the rank realized
or just narrated? is the enabling lemma non-circular?" The user dislikes being
flattered and corrects analysis directly — surface your own over-claims.

The recurring agent failure mode (correct it proactively): stating
- "the graph is acyclic" as if it proved a rank,
- "I see only these two channels" (necessity) as if it were generation,
- "splitting collapses the term" (intuition) as if it were a lemma.
Each of these needed the user to force the upgrade. Don't make them.

---

## 1. Theory carrier vs verification scaffold (decide FIRST, never conflate)

Pick the **intrinsic object** as the paper's theoretical carrier; the
coordinate / coefficient model is Lean scaffold only.

- Intrinsic: an operator on End/Hom, a submodule, a kernel of a canonical map.
  Reviewer-proof: "dim = N" reads as a structural invariant of End(·).
- Scaffold: a function-space coordinate model (`coeff : … → F`). Fast for Lean
  (support-disjointness, mechanical `split_ifs`), but a coordinate "dim = N"
  reads to a referee as a **basis artifact**.
- Connect them with an explicit **lift / factorization lemma**
  (coordinate model ≅ intrinsic object restricted to the relevant maps).
- Rule: Lean may stay in the scaffold; the paper MUST be lifted to the intrinsic
  carrier or it looks coordinate-dependent. Do not mistake Lean implementation
  convenience for the theoretical carrier (a real mistake made and corrected).

## 2. Lock the theorem statement as immutable BEFORE further work

- Pin **every operator's domain and codomain explicitly.** Ambiguous domains are
  the #1 source of a self-contradictory main theorem.
- **Self-consistency probe:** push a distinguished element through the statement.
  Example: for Φ(T)=2T([x,y])−[Tx,y]−[x,Ty], compute Φ(Id)=0 ⇒ Id∈ker(Φ).
  If the statement also says dim ker = N, ker is a complement of F·Id, and
  Im⊆I, then Id∈ker contradicts all three. The fix is a domain restriction
  (Φ on Hom(N,I), not Hom(N,N)), which excludes Id and makes the statement
  internally consistent. A 30-second probe catches a fatal record-locking error.
- Give a **structural reason** for each lock (closure / splitting / CE
  consistency), never "it's more convenient."
- Once locked, write it to an immutable doc (e.g. FINAL-THEOREM.md) and treat it
  as axiomatized; further work may not silently change it.

## 3. The ladder — drive every claim to the top before finalizing

```
intuition / "geometrically correct"
      │
structural implication / observation     "I see only these constraints"
      │
NECESSITY                                 "a nonzero X ⇒ must use channel Y"
      │
GENERATION / span                         "these channels GENERATE the whole
                                           row/constraint space — no hidden,
                                           no mixed-source rows"
      │
NON-CIRCULAR CLOSURE                       each enabling lemma is proved from a
                                           strict SUBSET of the constraints, so
                                           it may classify the FULL set w/o
                                           circularity
```

Concrete upgrades that were demanded (template for the class):

- **necessity → generation.** "Only probes A,B touch the constrained
  coordinates" (necessity) is NOT "A,B generate the entire constraint row space."
  To close the gap, also show the LEFT/other terms contribute nothing: e.g. a
  splitting bracket always lands in width≥2, and a separate **Stability Lemma**
  shows width≥2 images have zero projection onto the constrained coordinates —
  so no mixed-source rows exist. Generation = (universal channel enumeration) +
  (every other contribution provably vanishes).

- **acyclic graph → explicit rank realization.** "The coupling graph is a
  forest" does NOT give the nullity: acyclic ≠ full-rank constraint matrix;
  coefficient coupling can hide rank deficiency. Upgrade to a **filtration /
  block decomposition**: exhibit the constraint matrix (in a support ordering)
  as identity blocks (unit-coefficient single-variable eliminations) ⊕ a few
  1-dimensional coupling blocks ⊕ a zero block on the free modes. Then
  rank is **additive over independent support strata** and computed exactly.
  Watch the **redundancy bookkeeping**: a coupling equation appearing in two
  probe families is ONE equation — double-counting shifts nullity by 1.

- **intuition → explicit non-circular lemma.** A "collapse" used to simplify the
  constraint classification must be proved from a strict subset of the
  constraints (e.g. half-Leibniz on a SINGLE splitting pair, per source,
  independently), so it can then be used to classify the FULL constraint set.
  State non-circularity and uniformity explicitly in the lemma.

## 4. Sequence by mathematical dependency, not by feature choice

Once the structure is locked you are in **finalization stage, not construction
stage.** Do not pick next steps like product features (A vs B). Order by
dependency: identify the ONE genuinely unclosed gap (here: the upper bound /
no-hidden-freedom) and close it first; demote already-closed parts (lower bound,
explicit basis/cocycles) to "reframe only," not new math.

## 5. Numerical checks are sanity/regression, NOT closure

Small-n nullspace/rank computations may confirm (a) coefficient completeness and
(b) no hidden mixed-support rows — i.e. they regression-test the generation and
stratification lemmas. They can **never** stand in for the ∀n rank theorem.
Do not run them as the "proof"; run them to catch a missed channel before
locking the lemma.

## 6. One-line generation lemma you will keep needing

> Every constraint row (over all bracket pairs) whose support meets the
> constrained coordinates has, as one of its two factors, a member of the finite
> probe set; rows with neither factor a probe project to zero. Hence the
> constraint row space is generated by the probe-indexed rows.

Proof shape: (universal structure-constant enumeration: each generator of the
small invariant subspace brackets to a 1-dimensional image, nonzero only at a
unique argument) + (a stability lemma killing the remaining term). This is the
"row-space factorization via structure constants" that converts a Lie-algebra
constraint problem into pure linear algebra.
