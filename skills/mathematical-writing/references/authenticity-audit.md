# Authenticity Audit: Why a Paper Doesn't Read Like a Mathematician

> Discovered 2026-07-09 during cold-read audit of a paper that passed all
> 12 standard checks. The standard audits (error patterns, negative space,
> sentence metrics) were necessary but insufficient. Below is the
> next-level analysis.

## The Core Problem

A paper can be technically correct, free of forbidden words, and
structurally sound — and still not read like a mathematician wrote it.
The failure is in VOICE, not in correctness.

## 8 Categories of Authenticity Failure

### 1. Meta-Commentary (MOST IMPORTANT)

The paper describes its own structure, importance, or methodology instead
of just presenting mathematics. Every sentence that says "this is
important" or "this step does X" is a tell.

EXAMPLES OF WHAT TO DELETE:
- "This identity is the central computational tool of the paper."
- "The dominant contribution to the rank deficit comes from..."
- "The proof is the technical core of the paper."
- "This step collapses the target space from N dimensions to 3."
- "This constraint is structurally essential: it couples variables..."
- "Overcoming this requires multi-equation syzygies."
- "The rightmost column exhibits a structural degeneracy..."

RULE: Present the Lemma, prove it, move on. Let the structure speak.
If you need to explain WHY something is important, you haven't organized
the paper well enough.

### 2. The "Organized As Follows" Paragraph

"This paper is organized as follows. Section 1... Section 2..."

Short section-outline paragraphs are actually common in published math
papers. Ou (2007) has one: "This paper is organized as follows." So does
KK (2023). The tell is not the presence of an outline paragraph — it's
whether the paragraph reads like a bullet-pointed PhD-thesis enumeration
vs. natural prose. Three sentences integrated into the intro flow are
fine. A block of "Section 1 does X. Section 2 does Y. Section 3 does Z."
with equal-length sentences is the tell. Vary the structure.

### 3. Triple Definitions (Redundancy)

The same object defined in intro, §1, and §3. Each section should assume
the previous was read. A mathematician trusts the reader's memory.

CHECK: For each mathematical object (N_n, 1/2-derivation, I ideal, chain
bridge formula, bracket formula), count its definitions. Any count > 1
is a markdown.

### 4. Misplaced Forward References

"Section 3 proves that im(φ)⊆I" appearing in the introduction or §1.
The introduction should NOT announce results that belong in later
sections. Forward references are acceptable ONLY within the same section
(e.g., "We will prove this in Lemma 3.2 below").

### 5. Weak/Imprecise Mathematical Verbs

| Avoid | Use |
|-------|-----|
| "yields" | "gives" or "=" |
| "a unique surviving term" | "only one term is non-zero" |
| "dispatch by the row index" | "consider cases according to" |
| "all diagonal entries are equal" | "is independent of (i,j)" |
| "collapse" (re: dimensions) | "reduce" |

### 6. Sketched Verifications

For constructed maps (ω₁, ω₂, ..., ω₅), the verification must be
complete enough that a referee doesn't need to fill in gaps. "Commuting
pairs contribute zero by (I)" is insufficient — it doesn't explain the
cancellation mechanism. If a verification is "straightforward," still
give the key equation that makes it work.

### 7. Inaccurate Cross-References

"E_{1,n} is central by (I)" — but (I) defines the ideal I. Centrality
of E_{1,n} is a Lie algebra property that follows from the bracket
formula, not from how I is defined. Cross-references must be logically
accurate.

### 8. Abstract Tells

- "We classify the space of 1/2-derivations" — classify the
  *derivations*, not the "space"
- "five local rigidity mechanisms" — not mathematical language
- "The proof has been formalized in Lean." — unusual in an abstract;
  move to acknowledgments or a footnote

### 9. Tutorial Voice — Over-Explaining Routine Computations

Discovered 2026-07-11: a paper can pass all forbidden-word audits and
still read like lecture notes rather than a journal paper. The tell:
bracket expansions, coefficient identifications, and vanishing-term
justifications are spelled out step-by-step, as if teaching a student
how to compute [E_{ab}, E_{cd}].

A mathematician reader knows how to expand brackets. The paper should
state RESULTS of such computations, not narrate the computation itself.

TELLS (delete on sight):
- "only the term E_{ab} of φ(E_{cd}) can produce the target (u,v),
  via [E_{ab}, E_{cd}] = E_{uv}, contributing π_{ab}" →
  "E_{ab} contributes π_{ab}"
- "all other terms have column ≠ v or row ≠ 1 and give zero" →
  delete entirely (reader knows other terms vanish)
- "because the index sets are disjoint" after stating a pair commutes →
  delete ("commutes" is sufficient justification)
- Detailed δ-symbol walkthroughs like "δ_{n,i}E_{i+1,i+1} -
  δ_{i+1,i+1}E_{i,n} = -E_{i,n}" →
  state the result: "gives −X at (i,n)"
- "No other component produces the target: the δ_{b,i}-term
  requires i+1=n, which is incompatible with u>i+1" →
  delete; "only the term E_{u,n} contributes" already implies this

TARGET COMPRESSION: the same subcase proof should typically compress
by 40–50% without losing ANY mathematical content. If a subcase is
29 lines, it can usually be 15; if 53 lines, it can be 26.

The test: read a subcase aloud. If it sounds like you're teaching a
first-year graduate student how matrix brackets work, it's tutorial voice.
A journal paper trusts the reader.

CONTRAST WITH ACCEPTABLE DETAIL: the Claim proof in the same paper
uses "gives π = 2X" — this is journal-level. It states the result of
applying the half-Leibniz condition without explaining how brackets
expand. Use THAT voice everywhere.

## Audit Procedure

After passing all standard audits (error patterns, negative space,
sentence metrics), perform a COLD READ:

1. Read the paper as if you are a hostile referee who has never seen
   this work before.
2. Flag every sentence that describes the paper's own structure,
   importance, or methodology (category 1).
3. Count definitions of each object (category 3).
4. Check every forward reference outside its own section (category 4).
5. Verify each constructed object has a complete verification, not a
   one-line sketch pointing in a direction (category 6).
6. Read the abstract aloud — does it sound like a mathematics abstract
   or a grant proposal?
7. Check for tutorial voice (category 9): for each subcase proof,
   count how many lines explain HOW a bracket expands vs. stating the
   RESULT. Target: ≤ 1 expansion-explanation per subcase. If > 1,
   the paper is teaching, not publishing.

## What This Is NOT

This audit does NOT check:
- Mathematical correctness of proofs (separate audit)
- Forbidden words (separate checklist)
- Sentence length / structure metrics (separate checklist)
- Negative space / what to delete (separate checklist)

This audit checks what those audits miss: whether the paper SOUNDS
LIKE A MATHEMATICIAN WROTE IT.
