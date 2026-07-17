# English Mathematical Paper Conventions

> Based on analysis of Kaygorodov-Khrypchenko (arXiv:2305.00727) as the style
> reference, plus lessons from debranding our own paper. These conventions apply
> to the English original and influence the Chinese translation.

---

## Core principle: a math paper reports results, not the journey

The reader is a peer mathematician, not a student. The paper states what was
proved, not how it was discovered or why each step was chosen.

## What a math paper should NOT contain

### Named proof techniques

KK never gives names to proof steps. He uses lemma numbers and equation
numbers. Never "by the chain bridge" or "by width reduction" — always
"by Lemma 2.1" or "by (5)".

### Source/target/width as informal terminology

KK writes "e_{ij}" and "(i,j)-entry", never "source (i,j)" or "target (u,v)".
KK writes "pairs with j-i=2", never "width-2 sources". Informal descriptors
that are not standard matrix-algebra terminology should be replaced with
direct index arithmetic.

### Numbered step frameworks

Never "Step 1: ... Step 2: ... Step 3: ... Step 4: ...". The structure
should be communicated by the lemmas themselves. A short narrative paragraph
at the start of a long proof section is acceptable — but it should say what
each lemma DOES ("Lemma 3.2 shows that im(φ) ⊆ I") rather than just listing
names ("Step 2: Image restriction").

### Meta-commentary about proof structure

Never "is analyzed through the following reductions", "we now turn to",
"having completed X we address Y". The mathematics speaks for itself.

### Orphan subsections for single equations

If a subsection contains only one equation and one sentence of explanation,
merge it into the preceding subsection. KK's subsections are broad (e.g.
"Upper triangular matrix algebra" covers notation, bracket, center, and
commutator all in one).

### Ornamental observations

Never include symmetry observations or "by the way" facts that are not used
in any proof. Example: listing all Dynkin involution symmetries when only
one is referenced.

### Unproven exceptional cases

If the Lean formalization covers n≥5 only, the paper should not claim
results for n=4 (even with "a direct computation gives..."). Either prove
it or delete it.

### Tutorial-voice markers

- "We now prove" → "We prove" (or just start)
- "We distinguish three cases" → "There are three cases"
- "We split according to whether" → "Split according to whether"
- "We determine which..." → "The non-zero coefficients are determined by..."

### Banned words (from prior audit)

hence, yields, vanish, belong to, one checks, recall that, note that,
observe that (except for genuine observations like "Observe that E_{k,k+1}
is not a commutator"), it is worth noting, as we shall see, we will return to.

---

## Positive conventions (from KK)

### Global condition declarations

Instead of repeating char≠2,3 and n≥5 in every lemma, declare once after
Theorem 1.1: "In what follows we work under the hypotheses of Theorem 1.1.
Thus charF≠2,3 and n≥5 throughout."

### Lemma-first structure

KK's paper flows: Lemma → Proof → Lemma → Proof. No strategy overview,
no section introductions, no "in this section we will...". The lemmas
themselves communicate the logical structure.

### Concrete index arithmetic

KK writes: "e_{ij}·vf(e_{ii}) = vf(e_{ii})(j,j)e_{ij}", "the (k,l)-entry".
Always direct index manipulation, never metaphorical terminology.

### Acceptable transitional language

KK uses: "We denote by...", "Let...", "Then...", "Thus...", "Observe that...",
"We are going to prove...", "Finally,...". These are standard mathematical
transitions, not tutorial voice.

---

## Debranding workflow

When auditing a paper for self-coined terminology:

1. Identify the reference paper(s) in the same subfield (KK for 1/2-derivations
   on triangular matrices, Ou-Wang-Yao for derivations on N(n,F))
2. Extract the reference paper's LaTeX source from arxiv
3. Search for patterns: section titles, inline named concepts, metaphors
4. Compare term-by-term: does the reference paper use this term?
5. Classify each term: standard / descriptive / self-coined
6. Replace self-coined terms with lemma/equation references or direct descriptions
7. Compile after each batch of changes
8. Cross-check against Lean formalization for condition consistency

## Style comparison metric

When comparing against a reference paper, quantify:
- "Thus" density (KK: 5 across whole paper; acceptable range: 5-30 depending on proof structure)
- "gives" density (KK: 1; ours: 18 — acceptable due to repetitive bracket-expansion proof structure)
- "We" density (KK: 10 across whole paper)
- Subsection count vs paper length
- Tutorial voice markers (enumerated above)
