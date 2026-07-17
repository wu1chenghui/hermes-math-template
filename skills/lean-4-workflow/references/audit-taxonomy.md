# Mathematical Trust Audit Taxonomy

> 2026-06-30: The user's preferred audit methodology.

## Core Principle
Distinguish "proving correctness" from "understanding why." The user values mathematical
reasoning over Lean verification. Prefer "proposition + one mathematical reason" format.

## Preferred Audit Sequence
1. **Freedom Audit** — classify every degree of freedom (zero / determined / free)
2. **Coverage Audit** — verify every object is classified exactly once
3. **Hidden Assumption Audit** — ensure every logical step has an explicit theorem
4. **Origin Audit** — identify the mathematical structure each lemma reflects
5. **Derivation Audit** — explain why the proof has this shape
6. **Discovery Audit** — reconstruct the obstacle → response chain
7. **Invariant Audit** — identify mathematical invariants distinguishing parameters

## What the User Rejects
- Overstated claims: "structural invariant" when only "structural origin" is established
- Premature "100% confirmed" — wait for rebuild verification
- "Irreducible representation decomposition" when only "orbit decomposition" is proven
- Adding more audits once Layers 0-3 are complete
- Claiming "probability is negligible" — use "substantially increased confidence"

## What the User Values
- Independent mathematical derivations (root system, Serre relations)
- "Natural basis" not "unique basis" (unless proven)
- Distinguishing "one interpretation" from "the only σ-equivariant decomposition"
- CE Propagation Theorem over 1000-line case dispatch
