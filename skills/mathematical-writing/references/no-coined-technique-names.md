# Don't Name Proof Techniques — Evidence from KK 2023

> Created 2026-07-11 from direct source-code comparison. Updated with debranding workflow.
> KK paper: Kaygorodov-Khrypchenko, "Transposed Poisson structures on
> the Lie algebra of upper triangular matrices", arXiv:2305.00727.

## The finding

Kaygorodov-Khrypchenko (2023) classifies 1/2-derivations on T_n(F) —
the same mathematical genre as our paper. Their LaTeX source contains
**zero coined technique names**. All references are by lemma number or
equation number.

By contrast, our v3 paper coins ~7 technique names used as section
titles and inline references.

## KK's section structure (all descriptive, no named techniques)

```
\section*{Introduction}
\section{Definitions and preliminaries}
\section{Transposed Poisson structures on the upper triangular matrix Lie algebra}
  \subsection{Upper triangular matrix algebra}
  \subsection{1/2-derivations of the upper triangular matrix Lie algebra}
\section{Transposed Poisson structures on the full matrix Lie algebra}
```

No "Chain Bridge", "Width Reduction", "Diagonal Propagation", etc.

## KK's lemma labels (internal only, never used as public names)

```
\label{glavlem}                     % "main lemma" — Czech? Internal.
\label{basic-properties-of-vf(e_ij)}
\label{vf(e_ij)-in-<e_ij>}
\label{vf(e_ij)(i_j)=const}
\label{af-is-halfder}
\label{descr-Dl(T_n(F))}           % "description of Delta"
```

These are programmer labels, not published technique names.

## KK's reference style (lemma numbers, not names)

```
"by Lemma 2.1"
"Substituting this into \cref{...}"
"the desired equality is obtained by..."
```

Never: "by the chain bridge" or "by diagonal propagation".

## Our paper's coined names (for comparison)

| Our coinage | Where used | KK equivalent |
|-------------|------------|---------------|
| "the chain bridge" | §1.2 title, inline x3 | Just \eqref{eq:chainbridge} |
| "width reduction" | §1.3 title, inline x1 | Lemma number only |
| "diagonal propagation" | §1.4 title, intro, §3 | Lemma number only |
| "image restriction" | §3.1 title | Lemma number only |
| "endpoint reduction" | §3.2 title | Lemma number only |
| "a/c-channel" | §3 inline x4 | "coefficient of E_{1,n-1}" |
| "the Observation" | §2, inline x5 | "the fact that..." |

## The rule

**If a concept is already expressed as a numbered lemma or equation,
do NOT give it a separate English name.** The lemma/equation number
IS the name. This includes:

- Subsection titles: use descriptive names, not technique names
  ("Reduction to adjacent sources" not "Width reduction")
- Inline references: "by Lemma 3.1" not "by image restriction"
- Metaphors: "the coefficient of E_{1,n-1}" not "the a-channel"
- Capitalized facts: "note that" not "the Observation"

## Debranding Workflow (executed on v3, 2026-07-11)

### Step 1: Audit
Search the paper for every coined technique name. For each, identify:
- Where it's used (section title? inline reference?)
- What it refers to (equation number? lemma number?)

### Step 2: Plan replacements
For each coined name, decide:
- Can it be replaced by a lemma/equation number?
- If it's a section title, what descriptive alternative works?
- If it's a metaphor, what direct description replaces it?

### Step 3: Execute by file
Work through files in dependency order (intro → prelim → sec2 → ...).
Patch each occurrence individually. Verify compilation after each file.

### Step 4: Run the grep scan
Final scan: `grep -n 'old-name-1\|old-name-2\|...' *.tex` across all
active source files. Zero hits = done.

### Step 5: Recompile
Full tectonic compilation. Verify 0 errors, PDF size unchanged.

### Files modified in v3 debranding

| File | Changes |
|------|---------|
| exercise-prelim.tex | 3 subsection titles + 1 inline ref |
| exercise-image-restriction.tex | 1 subsection title + 3 inline refs + 1 label rename + 1 redundant text fix |
| exercise-endpoint.tex | 1 subsection title + 2 inline refs (a/c-channel) + 1 label ref update + 1 diagonal propagation ref |
| exercise-intro.tex | 1 route-map sentence rewritten |
| exercise-sec2.tex | 1 "Observation." paragraph + 5 "By the Observation" refs |
| exercise-strategy.tex | 3 Step X renaming |
| (label rename) | eq:channeldecomp → eq:coeffdecomp in 2 files |

### Common pitfall: LaTeX backslash escaping
When patching lines containing `\neq`, `\in`, `\ref`, etc., the patch tool
may double-escape backslashes. Always verify with `read_file` or
`python3 -c "print(repr(line))"` after patching. If the file shows `\\\neq`
(two literal backslashes), fix with `sed -i 's/\\\\neq/\\neq/g'`.
