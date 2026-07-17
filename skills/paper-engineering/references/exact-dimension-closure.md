# Closing an exact-dimension / classification theorem (paper-level)

Methodology for proving `dim V = N` (or classifying a space) rigorously on
paper, distilled from a multi-round referee-tightening of the N_n
half-derivation upper bound (2026-06-22). This is about the DERIVATION-time
rigor ladder, complementary to the post-hoc audits in the main SKILL.

The user drives this as a hostile referee and **will not accept "structural
inference" / "intuition" as a closed proof.** Climb the ladder proactively.

---

## 1. Anatomy: split into lower + upper bound, do the LOWER first

An exact-dimension theorem is two inequalities with very different character:

- **Lower bound `dim ≥ N` = an algebraic injection.** Exhibit N explicit
  independent elements via an *evaluation morphism* `ev : V → ∏ coords`,
  `T ↦ (T(e_1), T(e_2), …)`, and prove **coordinate injectivity** on the
  generators (`ev(T)=0 ⟹ T=0`). This is LOCAL, low-risk, and the right first
  step — it is the "safe anchor" that makes the theorem half-proven. In a
  formalization it is the most stable piece to do first.
- **Upper bound `dim ≤ N` = no hidden freedom.** This is the genuinely hard
  part: prove there is NO extra solution beyond the exhibited generators.
  Almost all the difficulty (and all the referee attacks) live here.

Sequence the work: lower-bound anchor first, upper bound last. Do NOT lead with
the upper bound.

---

## 2. The rigor-escalation ladder for the upper bound

Each rung was a real gap the referee forced. Skipping any one leaves an
informal proof masquerading as closed.

1. **Graph / "acyclic" intuition is NOT a rank proof.** An acyclic coupling
   graph guarantees an elimination ORDER, not full rank. Coefficient coupling
   can hide rank deficiency. You must produce an explicit constraint matrix and
   realize its rank (e.g. identity blocks ⊕ a 1-dim coupling block ⟹ rank
   additive). "Acyclic forest" should end up a *corollary*, not the argument.

2. **Necessity ≠ generation.** Showing "only these channels CAN appear"
   (support restriction) is weaker than "these channels GENERATE the entire
   constraint/row space". A row could have mixed support, or a hidden row could
   live outside your generators. You must prove **row-space factorization**:
   every constraint row is a linear combination of the identified generators.

3. **Downgrade "structural inference" to an explicit lemma.** Any step phrased
   as "by the structure this collapses…" must become a named lemma with a
   proof. Filtration-stability lemmas ("the operator sends filtration level ≥2
   into the lowest layer") are the usual shape.

4. **Non-circularity is a META-LEVEL justification, not proof text.** When a
   stability lemma is used to classify the very constraints it came from,
   referees flag dependency ambiguity. State it explicitly:
   *"The proof uses only the defining relations of ker(Φ) and a single
   application on one decomposition; it does not use any collapse-classification
   theorem as a premise. Hence it is independent of the collapse lemma."*
   Phrase it as "uses only X + one application", never "non-circular because we
   only use splitting".

5. **Numerical checks (small n) are sanity / regression ONLY.** Useful to
   confirm completeness of a case enumeration or catch a missed channel — never
   admissible as rank/dimension/closure proof. Don't let a numerical pass stand
   in for the structural argument.

---

## 3. The structural engine: a 1-dimensional derived image

The lever that made the whole upper bound tractable: the relevant substructure
`I` had a **1-dimensional derived image**, `[I, g] = F·e` (one direction).
Consequences to exploit:
- Every bracket of an `I`-valued map lands in `F·e` (1-dim), so a derivation-
  like operator's defect on width-≥2 inputs collapses to that single direction
  (a clean *filtration-stability* lemma).
- The constraint enumeration becomes a **universal ∀ bracket-table**: each
  generator of `I` has a UNIQUE nonzero bracket partner. This upgrades
  "two channels observed" to "two channels, by complete enumeration" — i.e.
  observation → theorem. Always look for such a collapsing engine before
  attempting brute combinatorics.

---

## 4. Lock the theorem statement BEFORE formalizing

Domain precision is load-bearing. A kernel/operator must specify its domain or
the statement self-contradicts. Example: `Φ(T) = 2T[x,y] − [Tx,y] − [x,Ty]`
taken on all of `Hom(g,g)` has `Φ(Id)=0`, so `Id ∈ ker Φ` — contradicting
`dim ker = N` and `Im(T) ⊆ I`. Restricting the domain to `Hom(g, I)` (maps into
the ideal) excludes `Id` and makes the splitting `V = F·Id ⊕ ker Φ` consistent.
Write the locked statement to a dedicated file, mark it immutable, and only then
formalize. Note any record that conflates `ker Φ` with the whole space is
superseded by the domain lock.

---

## 5. Theory carrier vs verification scaffold (two layers)

For the journal version, the intrinsic carrier is the operator space
`Hom(g, g)` / `End(g)` (so dim is an intrinsic invariant, the ideal is a real
algebra ideal, the operator is the canonical Leibniz-defect). A coordinate /
coefficient-function model is fine as a **Lean verification scaffold** but must
be lifted to the intrinsic carrier in the paper (a `structural lift lemma`:
coeff-model kernel ≅ intrinsic kernel), or the result reads as
coordinate-dependent. Keep the two layers explicitly separate; do not let the
scaffold become the theoretical endpoint.
