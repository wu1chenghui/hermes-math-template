# Paper-audit before a structure-decomposition / final-assembly step

Use BEFORE writing any Lean for a step that *looks* like packaging on top of a
closed sub-result: `V = F·e ⊕ W`, `dim V = 1 + dim W`, `ker(trace) = W`, "centered
⟹ in the ideal", etc. These steps usually hide **exactly one** real theorem; the
audit's job is to find it, confirm it's true, and confirm it's reachable — so the
work doesn't silently regrow into a second mini-DAG.

## 1. Isolate the minimal mathematical core

Decompose the target into "new mathematics" vs "formal transport". For
`V = F·e ⊕ W`, `dim = 1 + dim W`:
- `∩ = 0` (`μ·e ∈ W ⟹ μ=0`)  → easy, transport.
- `finrank = 1 + finrank W`     → easy, transport (`finrank_sup`/direct-sum).
- lower bound `dim V ≥ dim W + 1`→ easy (W's generators + e, e ∉ span(them)).
- **spanning / containment** `∀ v, v − λe ∈ W` → **this is the one real theorem.**
Name it in the design doc; state "everything else is transport" so future readers
know there is exactly one math node.

## 2. Verify every reused lemma is axiom-clean (`lean_verify` = full dep audit)

`lean_verify <FullyQualified.thm>` returns the theorem's **entire axiom closure**.
- `axioms = []` (purely constructive) or `{propext, Classical.choice, Quot.sound}`
  with **no `sorryAx`** ⟹ the theorem transitively depends on **zero sorry** — even
  sorries living in the SAME file (in other theorems) cannot taint it.
- This is strictly stronger than reading the proof body. Do it before depending on a
  lemma that sits in a sorry-laden / partially-falsified legacy file (you can import
  the file and use the clean lemma; just never touch the broken siblings).

## 3. Computationally validate the structural claim (before Lean)

Build the constraint system (e.g. half-Leibniz / cocycle equations) over ℚ for small
parameters, compute the nullspace, and check the claim numerically:
- nullity = the expected dimension (re-confirms the headline theorem cheaply);
- the claimed support containment holds (e.g. "centered solutions' off-diagonal
  support ⊆ ideal I" → the ∉I support in every basis vector is empty).
A `fractions`-based RREF in `execute_code` handles n≈5–8 in seconds; no sympy needed.

## 4. Exception-set closure (the make-or-break for split/width inductions)

If the proof is a split/width induction with an "exception set E" (targets ALLOWED
to be nonzero), the induction `IH` typically concludes `0` only for `∉E` targets.
For it to close, **the complement `∉E` must be closed under the split-children map** —
a `∉E` parent must only produce `∉E` children. Verify by enumerating the children
(`coeffOf_source_exists`-style 4-way split) over all `∉E` off-diag targets for small
n; a single `∉E → ∈E` leak means the induction breaks and E is wrong.

### "Wrong exception set, not wrong technique"

When an induction-based classification hits an irreducible gap / accumulates sorries
in a base case, **suspect the exception set is too small before suspecting the
technique.** (Canonical failure mode: a project proved `dim = n` with exception
`{(1,n)}`; the truth needed `I = {(1,n-1),(1,n),(2,n)}` — three boundary cocycle
targets. The old base-case lemma was trying to prove FALSE statements
`coeff(adjacent; (1,n-1)) = 0`, hence unavoidable sorries. The fix was the exception
set, and the base case became provable once it only asserted true facts.) The right
exception set is the **minimal closed** one: closed under split (§4), minimal because
each member is realised as a nonzero basis cocycle.

## 5. Pin a base/leaf coefficient with COMMUTING partners, not parent-split

To pin a coefficient at a source that can't be split (an adjacent/leaf source), do
NOT reverse-solve from a parent split — that is **circular** (the parent's value is
itself proven by the same induction descending back through the leaf).
Use a **commuting partner** instead: pick a partner source whose Lie bracket with the
leaf vanishes (`bracketSource = 0`), so half-Leibniz collapses to a **pure linear
relation** among coefficients (`bracketIdentity = 0`) — no width reduction, acyclic.
Expand it with the structural `bracketIdentity_eq_expanded` (which does NOT need any
`I_filtered`/filter hypothesis), so the partner terms are non-leaf coefficients the
main induction already kills. (This is the F1/F2/coupling pattern; the commuting-
vanishing lemma is unconditional, unlike the filtered canonical-form lemmas.)

## 6. The "carrier includes a filter" trap

A `Submodule` carrier defined with an extra predicate (e.g. `I_filtered`,
`IsValidSupp`) may **exclude the natural unit element** `e` of the decomposition.
Example: an identity map `Id` has diagonal targets `(i,j)` that fail an
`I_filtered`-style predicate, so `Id ∉ W` even though `Id ∈ V`. Then `V ⊋ W` and the
full space must DROP that filter (keep only the genuine half-Leibniz + validity
conditions), with `W` = the filtered subspace. Always check the unit element's
membership explicitly, and beware unguarded `coeff_Id`-style defs that are also not
valid-supported (nonzero on illegal indices) — define a guarded `e_valid`.

## Workflow note

Land the design as a SEPARATE `.md` (not the global theory doc), get the referee's
"open-work" sign-off, THEN write Lean. The audit's deliverables (axiom-clean
confirmation + closure check + named minimal core) ARE the open-work permit.
