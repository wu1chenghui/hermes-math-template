# Assembly-Interface Skeleton Probe (before mass-implementing leaf lemmas)

## The lesson — hypothesis-circularity

A theorem that **establishes** property `P` must NOT take `P` as a hypothesis —
else it is vacuous. Corollary: any leaf lemma whose proof **assumes** `P` cannot
be plugged into the `P`-establishing theorem.

E-project instance (D3 `imageContainment`, the `HalfDer = F·Id ⊕ kerΦ` step):
- `I_filtered coeff hn ≡ ∀ i j u v, coeff i j u v ≠ 0 → (u,v) ∈ I_target_set hn`,
  i.e. "image ⊆ ideal I", i.e. "every ∉I coeff = 0" (PhiOperator.lean).
- `imageContainment` (feeding the §D centering) is what *establishes*
  I-containment, so it MUST be stated **without** `hI : I_filtered`. With `hI`,
  the goal `coeff(∉I) = 0` follows from `hI` directly → vacuous, all leaf
  machinery dead.
- But the boundary leaves kill their wide-`∈I` residual via `width_stability_a/c`,
  which **require** `hI`. So those leaves cannot plug into the hI-free theorem.
- Extra trap: the residual is an `∈I` target with target-width LARGER than the
  proven coeff, so it is also OUT of the target-width `ih`'s reach.

This circularity is INVISIBLE at the leaf level — each leaf compiles fine carrying
its own `hI` — and only surfaces at assembly. (The zero-residual / hypothesis-free
leaf class plugs in fine; the hypothesis-dependent classes do not.)

## The probe — catch it BEFORE enumeration

Before writing the per-case enumeration / mass leaf set, build a `<thm>_skeleton`
with the FINAL (hypothesis-free) statement and:
1. set up the real induction (e.g. `induction' hlen : v - u using
   Nat.strong_induction_on with d ih generalizing i j u v`, mirroring the
   sorry-free precedent — confirm `generalizing` reverts the new exception
   hypothesis and the `ih` gains the matching argument);
2. wire ONE representative of each leaf class + the recursive branch
   (here `coeffOf_source_exists` + `ih`);
3. leave structural obligations as `sorry`, but confirm every hypothesis THREADS.

The recursive/nonadjacent branch is typically hypothesis-free and clean; the
mismatch (if any) surfaces exactly at the base-case leaf dispatch. Do NOT land the
sorry-skeleton into the sorry-free active file — it is a `lean_run_code` probe.
Landing a statement that the resolution will rewrite is wasted work.

## Resolutions when it bites

- **(R3, usual fix)** Bundle the dependent sub-fact into a JOINT strong induction
  so it is discharged internally with no external hypothesis (here: prove
  "∉I coeffs vanish" AND "wide ∈I channel-collapse" in the SAME induction; the
  residual is killed by the collapse clause). Changes the theorem statement.
- **(R2)** Re-prove the dependent lemma `ih`-grounded / hypothesis-free (feasible
  only if its use of the hypothesis is replaceable by the `ih`).
- **(R4)** Re-examine whether a different leaf construction lands the residual on a
  smaller-measure object the `ih` already covers (often impossible if the residual
  is structurally in the exception set).

Relation to `mechanism-probe-before-architecture-freeze.md`: that probe validates a
single leaf's MECHANISM; this probe validates the ASSEMBLY interface (induction +
leaf API + hypothesis threading) across the whole theorem. Run the mechanism probe
per edge-type first (Gate 1/2), then this assembly probe (Gate 3) before
committing to the full enumeration. Leaf recipe:
`lean-4-proof-writing / references/commuting-partner-bracket-vanishing.md`.
