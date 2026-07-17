# Blind-Review Style Audit — Ou/KK vs Our Paper

> Methodology developed 2026-07-12 during a systematic comparison of our paper
> against Ou-Wang-Yao (2007) and Kaygorodov-Khrypchenko (2023).

## The Question

If our paper were presented alongside Ou (2007) and KK (2023) for blind review,
would a reviewer think it was written by the same author(s)?

## Audit Dimensions

### 1. Roadmap Paragraphs

**Ou**: Zero roadmap. Theorem stated, immediately moves to §1.
**KK**: At most 1 sentence: "The paper is organized as follows."
**Us (v3)**: 7-line roadmap with colon-enumerated list of 4 items:

```
In Section 1 we fix notation and establish the basic structural facts:
a basic bracket identity, a reduction to adjacent basis elements,
the equality of diagonal entries, and the decomposition...
Section 2 constructs... Section 3 proves...
```

**Verdict**: High-impact tell. A colon-enumerated roadmap with 4+ items is a
modern thesis convention, absent from the Ou/KK tradition.

### 2. Case-Label Nesting Depth

**Ou**: Uses (1)(2)(3) numbered items within proofs. At most 2 levels.
**KK**: Similar — flat or 2-level structure.
**Us (v3)**: Three-level nesting in image-restriction proof:

```
Case 1: v < n.              ← Level 1: bold Case header + \medskip
  u ≥ i.                    ← Level 2: bold condition + \smallskip
  u < i, v = i+1.           ← Level 2
  u < i, v ≠ i+1.           ← Level 2
Case 2: v = n.              ← Level 1
  (i) u = i.                ← Level 3: bold (i)(ii)(iii)(iv) + \smallskip
  (ii) u = i+1...           ← Level 3
  (iii) u < i, i = n-1.     ← Level 3
  (iv) u > i+1.             ← Level 3
```

**Verdict**: High-impact tell. The bold labels + `\smallskip` + 3-level nesting
reads like lecture notes, not a journal paper.

### 3. Explanatory Parentheticals

**Ou**: 0 explanatory parentheticals. All parentheticals are math notation.
**KK**: 0 explanatory parentheticals. Single-word labels only (`bilinear`, `non-orthogonal`).
**Us (v3)**: 7 explanatory parentheticals across proofs:

```
1. (using $d_3=d_1$, $d_4=d_2$ from above)       ← prelim
2. (only the bracket-right term contributes, via... ← image-restriction
3. (only $c_kE_{2,n}$ interacts with $E_{12}$)    ← endpoint
4. (via $[E_{1,n-1},E_{n-1,n}]=E_{1,n}$)         ← endpoint
5. (trivially if $u>i$, by Lemma 3.1 applied to
    $(v,v+1)$ if $u=i$)                           ← image-restriction
6. (the $E_{i+1,n}$-component of $\varphi(E_{i,u})$
    cancels the $E_{u,n}$-component of ...)        ← image-restriction
7. ($\operatorname{char}F\neq3$)                  ← image-restriction
```

**Verdict**: High-impact tell. Even after finding and deleting 4, the remaining 3
prove that automated regex scanning is insufficient for parenthetical detection.

### Detection Methodology Pitfall

Automated regex scanning with prose keywords (`using`, `only`, `via`, `trivially`,
`since`, `by`) missed 3 of 7 parentheticals because:

1. Parentheticals span line breaks (regex `[^)]*` doesn't cross lines)
2. Diverse vocabulary: `cancels`, `applied to`, `component` — no single keyword set covers all
3. `\operatorname{char}F\neq3` looks like math notation but functions as an explanatory reminder

**Correct workflow**: After automated scan, do a manual line-by-line re-read of
every proof. Read each parenthetical and ask: "Would Ou write this here?"

### 4. Global Hypothesis Re-Declaration

**Ou**: States conditions in theorem. Never re-declares them outside.
**KK**: Same.
**Us (v3)**:

```
In what follows we work under the hypotheses of Theorem~1.1.
Thus char F≠2,3 and n≥5 throughout.
```

**Verdict**: Medium-impact tell. Tutorial convention — telling the reader what
the standing assumptions are.

### 5. Pattern Language

**Ou/KK**: Never abstract proof steps into named patterns.
**Us (v3)**: "The same three-equation pattern as for u=1 applies, with the row
shifted from 1 to 2."

**Verdict**: Medium-impact tell. Meta-commentary on proof structure is a
pedagogical device absent from the reference papers.

## Summary of Tells (by impact)

| # | Tell | Impact | Location |
|---|------|:------:|----------|
| 1 | 7-line colon-enumerated roadmap | High | intro:29-35 |
| 2 | 3-level case nesting with bold labels | High | image-restriction:60-133 |
| 3 | 7 explanatory parentheticals | High | across 3 files |
| 4 | Global hypothesis re-declaration | Medium | intro:26-27 |
| 5 | Pattern-language meta-commentary | Medium | image-restriction:30 |
