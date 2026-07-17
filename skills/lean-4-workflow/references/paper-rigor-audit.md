# Paper Writing Rigor Audit

> When converting Phase design documents into a publishable paper, audit each
> section for specific completeness criteria. The §2–§7 audit of the HalfDer
> paper found 18 issues across 6 sections.

## Audit criteria per section

For each section, check:
1. **No undefined objects.** Every symbol/space/map is defined in this section or a prior one.
2. **Theorem dependencies closed.** Every proof references only prior theorems/definitions.
3. **No "obviously/clearly" without justification.** Every claim has an explicit reason.
4. **Proofs are complete.** The last line actually proves the stated result.
5. **No Phase references.** Phase document labels must not appear in the paper.

## Common issues found

| Type | Example |
|------|---------|
| Notation error | `Hom^{N_n}` (means N_n→Hom) used for `(Hom→N_n)` |
| Circular count | `|B_cand| = dim(Hom)-(n+5)` stated before main theorem |
| Sketch proof | "direct computation" without showing the computation |
| Phase reference | "Phase~C, \S C.4" appearing in paper |
| Missing definition | "centered" used in prose without formal definition |
| Wrong computation | `φ_F(e_{n+4}) = -2` when actual expansion gives 0 |

## Proof status labels

Use exactly three: **Established**, **Conjectured**, **Conditional**.
Never: "Key finding," "Expected," or percentages.

## Proof debt tracking

Maintain a PROOF_DEBT.md with columns: ID, Section, Issue, Affects main theorem?, Status.
Resolve all "Yes" items before declaring the proof complete.
