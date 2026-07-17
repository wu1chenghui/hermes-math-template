# Mathematical Paper Writing — Style Conventions

> Reference paper: Kaygorodov–Khrypchenko (2023), "Transposed Poisson structures
> on the Lie algebra of upper triangular matrices" (arXiv:2305.00727).
> This document captures style lessons from auditing a classification paper
> against KK's conventions.

## 1. No Self-Coined Technique Names

KK names **zero** proof techniques. Every reference is by lemma number or
equation number. Never give a proof step a formal name.

| ❌ Avoid | ✓ Use |
|----------|-------|
| "by the chain bridge" | "by \eqref{eq:chainbridge}" |
| "by width reduction" | "by Lemma~\ref{lem:width}" |
| "the diagonal propagation" | "the equality of diagonal entries" |
| "Image restriction" (subsection title) | "Restriction to the ideal I" |
| "a-channel analogue" | "the same computation for the $E_{1,n-1}$-coefficient" |

**Detection method**: Compare subsection titles and inline references against
the reference paper. If your paper has named subsections for techniques that
the reference paper describes with plain text, you're coining terms.

## 2. No Informal Index Descriptors

KK never uses "source", "target", or "width" for matrix indices. Write
direct index arithmetic.

| ❌ Avoid | ✓ Use |
|----------|-------|
| "source (i,j)" | "$E_{ij}$" or "$(i,j)$" |
| "target (u,v)" | "$(u,v)$" (context makes role clear) |
| "adjacent source" | "adjacent basis element $E_{i,i+1}$" |
| "width $j-i-1$" | "difference $j-i-1$" |
| "width-2 sources" | "pairs with $j-i=2$" |

Even when used as informal descriptors (not formally defined terms), their
heavy use creates a nonstandard vocabulary. KK writes "e_{ij}" and
"the (u,v)-entry" — always standard matrix terminology.

## 3. Global Condition Declaration

Rather than repeating char≠2,3 and n≥5 on every lemma, declare once after
the main theorem and let all subsequent statements inherit the conditions.

```tex
\begin{theorem}\label{thm:main}
For $n\ge5$ and $\operatorname{char}F\neq2,3$,
$\dim\Delta(N(n,F))=n+5$.
\end{theorem}

In what follows we work under the hypotheses of Theorem~\ref{thm:main}.
Thus $\operatorname{char}F\neq2,3$ and $n\ge5$ throughout.
```

Then individual lemmas drop redundant conditions. If a lemma genuinely needs
a different condition (e.g., only n≥3), state it explicitly.

## 4. No Tutorial / Lecture-Note Voice

Signs your paper reads like lecture notes, not a research paper:

| Marker | Example | Fix |
|--------|---------|-----|
| Numbered proof steps | "Step 1: ..., Step 2: ..." | Narrative flow: "We first show... From this we deduce..." |
| Meta-commentary | "is analyzed through the following reductions" | Just state the lemmas |
| Tutorial openings | "We now prove..." | "We prove..." or start directly |
| Bullet-point strategy overviews | List of 4 Steps | Paragraph that tells what each lemma DOES |
| Explanations of WHY | "The idea is..." | Delete; the math speaks for itself |

KK opens his proof sections by stating the first lemma. No overview paragraph,
no numbered steps, no "the proof proceeds as follows." The lemmas themselves
provide the structure.

### Acceptable overview vs unacceptable

**Unacceptable** (tutorial voice):
```
We now prove dim ≤ n+5. The centered part is analyzed through
the following reductions.

Step 1: Equality of diagonal entries.
Step 2: The image lies in I.
Step 3: Constraints on the endpoints.
Step 4: Parameter tally.
```

**Acceptable** (narrative flow):
```
We prove dim ≤ n+5. Let φ = s·id + φ₀ by Lemma 1.3.
Lemma 3.2 establishes im(φ₀) ⊆ I, reducing each φ₀(E_{i,i+1})
to a combination of E_{1,n-1}, E_{1,n}, E_{2,n}. Lemma 3.3
constrains the coefficients, leaving n+4 parameters for φ₀
and n+5 overall.
```

The acceptable version still orients the reader but does so through
**what the lemmas achieve**, not through a numbered lesson plan.

## 5. Auditing Checklist

When checking a paper against a reference (like KK):

- [ ] All subsection titles use descriptive language, not named techniques
- [ ] All inline references use lemma/equation numbers, never named techniques
- [ ] No "source", "target", "width" as matrix index descriptors
- [ ] Conditions declared once globally, not repeated on every lemma
- [ ] No numbered "Step 1/2/3" bullet points
- [ ] No "analyzed through the following...", "we now prove...", "the idea is..."
- [ ] "Thus" not overused (KK: 5 in whole paper; if >>10, vary with "Hence"/"Therefore" or restructure)
- [ ] "gives" not overused (KK: 1; if >>10, vary with "we obtain", "results in", "it follows that")
- [ ] Banned words checked: hence, yields, vanish, belong to, one checks, recall that
- [ ] Compare word frequency against reference paper for calibration
