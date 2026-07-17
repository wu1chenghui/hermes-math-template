# Translation Specification — 10 Principles

> Canonical specification for mathematical paper translation (English → Chinese),
> as ratified by the user. This is the authoritative reference for how translation
> must be done — deviate from it only with explicit user permission.

---

## Principle 1: Chinese version always maps to English version

The Chinese version must satisfy:

> Any Chinese sentence can be uniquely mapped back to the original English sentence.

Never:
- Reorder proofs
- Add explanations
- Rewrite proof logic for Chinese fluency
- Insert new mathematical exposition

Reason: if the English version changes a sentence, the Chinese version can be directly synchronized. Two independently-evolving versions will inevitably diverge.

## Principle 2: Mathematical content is untouched

Keep all structural elements intact:
- Definition, Lemma, Theorem, Proof, Remark, Corollary
- All formulas (do NOT rewrite formulas in Chinese prose)
- All numbering (equation tags, section numbers, theorem numbers)
- Paragraph positions should be preserved

Example violation:
> English: "We first prove..."
> Bad: "为了说明这个事情，我们现在先来证明……" (rewriting)
> Good: "我们首先证明……" (translating)

## Principle 3: Unified glossary — one term, one translation

Before translating any section, establish a complete glossary. Every English term maps to exactly one Chinese term. Never use multiple Chinese translations for the same English term.

Example violation:
> "image restriction" → "像约束" / "像限制" / "值域限制"

## Principle 4: Parenthesize English on first occurrence

On first use: show English in parentheses, then use only Chinese:
> 链桥（Chain Bridge） → later just "链桥"

This follows standard Chinese mathematical paper conventions.

## Principle 5: Do NOT translate Lean identifiers

Lean theorem/lemma names stay as-is with backticks:
> `coeffOf_cond`, `finrank_halfDer`

Do NOT coin Chinese names for them.

## Principle 6: Translate section titles, preserve numbers

Section headings are translated; numbers stay exact:
> `5.4 Image Restriction` → `5.4 像限制`

## Principle 7: Formulas are NEVER modified

Formulas are identical in both versions. Body text may explain, but display math is untouchable:
> `Im(T₀)⊆I` stays `Im(T₀)⊆I`

## Principle 8: No narrative "polishing"

Translation is about accuracy, not elegance:
> "We now prove..." → "现在证明……"
> NOT: "接下来我们将详细证明……"

## Principle 9: Verify each section immediately after translation

Workflow: English §N → Chinese §N → line-by-line comparison → confirm fidelity → §N+1

Do NOT translate all sections before checking.

## Principle 10: Chinese version does not participate in mathematical revision

The English version is the authoritative paper. Revisions:
1. Revise English first
2. Confirm changes
3. Synchronize Chinese

Never revise both versions independently.

---

## Recommended Workflow

1. Establish glossary (~100 core terms)
2. Translate section by section (§1 → §8)
3. After each section: fidelity check (no additions, deletions, logic changes, formula/numbering changes)
4. Final pass: global terminology consistency audit

---

## Fidelity Check Protocol

Per section, verify:
1. Sentence count: Chinese ≈ English (±1)
2. Formula count: identical `\[...\]` and `\begin{equation}` blocks
3. Theorem environments: identical counts
4. Labels: all `\label{...}` and `\ref{...}` preserved
5. Citations: all `\cite{...}` preserved
6. Index ranges: all subscript/superscript ranges preserved
