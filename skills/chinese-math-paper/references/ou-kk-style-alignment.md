# Ou/KK Style Alignment — Pre-Translation Checklist

> Distilled from the 2026-07-12 paper-v3 → paper-zh translation session.
> After compressing the English paper, run this checklist to align with
> Ou-Wang-Yao (2007) and Kaygorodov-Khrypchenko (2023) writing style.

## Reference Papers

- **Ou (2007)**: "Derivations of the Lie algebras of strictly upper triangular
  matrices over a commutative ring", LAA, 6 pages. Terse, no exposition,
  no named techniques, no roadmap, no parenthetical explanations.
- **KK (2023)**: "Transposed Poisson structures on the upper triangular matrix
  Lie algebra", arXiv:2305.00727, 12 pages. Slightly more structured than Ou
  but still terse. Descriptive section titles, no coined technique names.

## Checklist (priority order)

### 1. Remove explanatory parentheticals

Ou/KK never use parenthetical explanations. Any `(...)` that explains
WHY something is true should be deleted — the surrounding sentence
must stand as a self-contained assertion.

**Pattern to find**: `(using ...)`, `(only ...)`, `(via ...)`,
`(trivially ...)`, `(the ... component cancels ...)`, `(char F≠3)`

**Scan command**:
```python
import re
prose_kw = [r'\busing\b', r'\bonly\b', r'\bvia\b', r'\bi\.e\.\b',
            r'\bsince\b', r'\btrivially\b', r'\bapplied to\b']
# Find parentheticals in non-math text matching these keywords
```

**Example fix**:
```
Before: gives π_{u,n}=0 (only the bracket-right term contributes).
After:  gives π_{u,n}=0.
```

### 2. Delete roadmap paragraphs

Ou's introduction has ZERO roadmap. KK has at most one sentence.
Any paragraph listing section contents with colons/numbers must go.

**Example**: Delete entire paragraphs like:
"In Section 1 we fix notation and establish the basic structural facts:
a basic bracket identity, a reduction to adjacent basis elements, ..."

### 3. Flatten case labels

Ou uses plain "If ..." / "Assume ..." sentences, never bold labels.
Convert all `\textbf{Case 1: v < n}`, `\textbf{(i)}`, etc. to plain
"If ..." paragraphs separated by blank lines.

**Before**:
```latex
\medskip\noindent\textbf{Case 1: $v<n$.}
Here $(u,v)\notin I$ means...
\smallskip\noindent\textbf{$u\ge i$.}
The commuting pair...
```

**After**:
```latex
If $v<n$.
Here $(u,v)\notin I$ means...

If $u\ge i$, the commuting pair...
```

Also fix cross-references: `Case~1` → `the case $v<n$`.

### 4. Compress global declarations

Ou states conditions in the theorem and assumes them. Avoid tutorial
declarations.

**Before**: "In what follows we work under the hypotheses of Theorem 1.1.
Thus char F≠2,3 and n≥5 throughout."

**After**: "Throughout, char F≠2,3 and n≥5."

### 5. Remove coined technique names

Replace ALL self-coined names with equation/lemma numbers:

| Forbidden | Replacement |
|-----------|------------|
| chain bridge | `\eqref{eq:chainbridge}` |
| width reduction | Lemma reference |
| diagonal propagation | Lemma reference |
| source/target/width metaphors | direct index notation |
| a-channel / c-channel | `$a_k$-coefficient` / `$c_k$-coefficient` |

### 6. Remove "pattern" language

Ou never describes proof steps as "patterns". Replace with direct
reasoning.

**Before**: "The same three-equation pattern as for u=1 applies,
with the row shifted from 1 to 2: [three equations]"

**After**: "If u=2, the same argument yields π_{2,i+1}=0."

### 7. Merge sub-case enumeration

When sub-cases are listed with "the following cases exhaust the
possibilities", merge into a single conjunction:

**Before**:
"If u < i and v ≠ i+1, the following cases exhaust the possibilities.
If u > 1, ... If u = 1, ..."

**After**:
"If u < i and v ≠ i+1, then for u > 1 ... while for u = 1 ..."

### 8. Remove conversational transitions

**Before**: "We now prove dim Δ ≤ n+5."
**After**: "We prove dim Δ ≤ n+5."

### 9. Align construction labels with reference format

If the reference paper (e.g., Ou) uses (A)(B)(C) lettered labels for
its standard-derivation construction, match the format. This creates a
subtle but effective visual alignment for readers familiar with the field.

**Before**:
```latex
\textbf{Trivial derivation.}
\textbf{Simple-root derivations.}
\textbf{Boundary derivations.}
```

**After**:
```latex
\textbf{(A) Trivial derivation.}
\textbf{(B) Simple-root derivations.}
\textbf{(C) Boundary derivations.}
```

Sub-labels under (C) — e.g., (i) ω₁ through (v) ω₅ — form a natural
two-level nesting and should be kept as-is.

## Pitfalls and False Positives

### Regex scans miss ~40% of parentheticals

Automated regex scanning for explanatory parentheticals is unreliable.
A scan with prose keywords (`using`, `only`, `via`, `trivially`, `applied
to`) will miss parentheticals that:
- Span line breaks (LaTeX `\n` in source)
- Use unanticipated phrasing ("component ... cancels", "force pairwise
  cancellation", `($\operatorname{char}F\neq3$)`)
- Hide in deeply nested case analyses

**Required workflow**: automated scan → manual line-by-line re-read of
every proof body. Read each `(...)` and ask "Would Ou write this?" If
the answer is no, delete it. Never absorb into the sentence — a `via`
clause without parentheses is still an explanatory aside and must go.

### Not every organizational phrase is a tell

Some English-to-Chinese-math-paper translation patterns that look
suspicious are actually standard mathematical English:

| Phrase | Status | Reason |
|--------|:------:|--------|
| "We split according to whether X or Y" | **OK** | Standard case-split introduction |
| "If X." as a standalone sentence | **OK** | Common in terse mathematical prose |
| "The same argument yields..." | **OK** | Standard abbreviation for identical reasoning |
| "We prove..." (no "now") | **OK** | Standard proof-transition language |
| "Thus" at 4-5 per 1000 words | **OK** | In normal range; only flag if >8/1000 |

**Actual tells** (must remove):

| Phrase | Fix |
|--------|-----|
| "We **now** prove" | "We prove" |
| "the following cases exhaust the possibilities" | Merge into "for...while..." sentence |
| "The same three-equation **pattern**" | "The same argument" |
| "In what follows **we work under**..." | "Throughout, ..." |

### Subsection granularity is NOT a tell

Ou (2007) has 0 subsections because his Preliminaries are trivial.
Our paper (5 sections, 7 subsections) is more granular because we have
more intermediate lemmas. Flattening subsections into continuous prose
would hurt readability without improving Ou-likeness — the difference
is in the mathematics, not the writing style.

## Verification

After all changes:
- [ ] `grep` for coined terms returns 0 in active .tex files
- [ ] Manual re-read of all proof bodies confirms 0 explanatory parentheticals
- [ ] No `\textbf{Case`, no `\textbf{(i)}` in proof bodies
- [ ] Construction labels use (A)(B)(C) format matching reference paper
- [ ] No "pattern", "exhaust the possibilities", "we now prove"
- [ ] Roadmap paragraph deleted
- [ ] `tectonic` compiles with 0 errors
- [ ] Page count stable (should not increase)

## Typical Results (paper-v3 session, 2026-07-12)

After compression + alignment:
- English: 610 → 508 lines (−102, −17%)
- Chinese: 571 → 492 lines (−79)
- Explanatory parentheticals: 7 → 0 (across 2 audit rounds)
- Coined terms: "chain bridge" → `\eqref{eq:chainbridge}` (all instances)
- Case labels: 9 bold labels flattened to plain "If..." sentences
- Construction labels: aligned to (A)(B)(C) format matching Ou (2007)
- Roadmap paragraph: deleted
- Global declaration: compressed to one line
- Pattern language: "three-equation pattern" → "same argument"
- Pedagogical phrases: "exhaust the possibilities" → "for...while..."
- Transition cleanup: "We now prove" → "We prove"
- Pages: English 5 (stable), Chinese 6 (stable)
- Lean audit: 16 paper claims → 16 Lean theorems, 0 sorries, conditions aligned

**Blind review result**: paper now passes as belonging to the Ou/KK tradition.
Remaining differences (subsection granularity, lemma count) are natural
consequences of mathematical content, not stylistic tells.
