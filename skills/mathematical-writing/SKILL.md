---
name: mathematical-writing
description: >-
  Write mathematical papers that read like professional mathematics, not
  development logs. Covers paper organization, motivation, English usage,
  notation, voice, AMS formatting, and modern 21st-century practices.
  Synthesized from Tao, Halmos, AMS, Conrad, Poonen, Serre, and Pak.
---

# Mathematical Writing — Principles

> **When to load**: translating a Lean proof into a paper, writing a new paper
> from scratch, revising a paper to match published conventions, or analyzing
> reference papers to extract writing patterns and sentence-level DNA for
> imitation. Load BEFORE `paper-engineering` or `chinese-math-paper` — this
> skill defines the writing quality standard those skills should produce.
>
> See `references/sources.md` for original research materials and full transcripts.
> See `references/error-patterns-checklist.md` for the pre-flight and post-audit checklists.
> See `references/negative-space-checklist.md` for what to DELETE from a draft.
> See `references/reference-paper-analysis.md` for the complete sentence-level
> writing DNA extracted from published derivation-classification papers
> (Ou-Wang-Yao 2007, Kaygorodov-Khrypchenko 2023) — including forbidden words,
> bracket calculation templates, two computational philosophies, and narrative arc.
> See `references/authenticity-audit.md` for the cold-read audit that catches
> what standard checklists miss — meta-commentary, "organized as follows"
> paragraphs, triple definitions, sketched verifications, and imprecise verbs
> that make a paper read like a development log rather than a math paper.
> See `references/polish-checklist.md` for the final-pass polish audit —
> subtle tells (internal code names, "one checks", non-standard case labels,
> repeated facts, filler sentences, "belong to", one-line lemma proofs) that
> survive standard audits but mark a paper as not-quite-professional.
> See `references/per-file-audit-template.md` for the systematic per-file
> audit methodology — A-group grep scans, B-group per-sentence checks,
> C-group per-paragraph deletion tests, with mandatory re-scan after all fixes.
See `references/paper-density-comparison.md` for the complete compression
methodology — four-step density diagnostic, three-tier verdict system
(Compress/Shorten/Keep), four-round execution workflow with concrete
before/after examples, Lean-verification safety step, typo-detection
patterns, and the cumulative 741 to 623 line transformation from 6 to 5
pages.
>
> See `references/bracket-computation-techniques.md` for non-obvious techniques
> chain-bridge + centeredness for diagonal elimination, commuting partner
> selection tables, 2x2 system patterns, and column-index induction).

> See `references/qualitative-reading-framework.md` for paragraph-level
> analysis that GREP cannot perform — the Q1-Q6 framework for studying
> reference papers' compression strategy, narrative arc, and emotional
> rhythm. Includes complete qualitative maps of Ou 2007 §3 and KK 2023
> Lemma 6-7.
>
> See `references/no-coined-technique-names.md` for the source-code
> comparison with KK 2023 that shows reference papers use zero coined
> technique names — all references are by lemma/equation number, and
> subsection titles are purely descriptive, never named after a technique.
>
> See `references/style-benchmarks.md` for quantitative cross-paper
> comparison of word frequencies (we, Thus, Hence, Therefore, gives,
> obtain) across KK 2023, Ghimire-Huang 2016, Yusupov 2025, and our paper.
> Use to calibrate whether a paper's style is in the normal range for
> derivation-classification papers.
>
> See `references/structural-anti-patterns.md` for the structural patterns
> that distinguish lecture notes from research papers: ornamental paragraphs,
> single-equation subsections, subsections named after proof methods,
> numbered strategy overviews, index-role descriptors (source/target/width),
> and human-readable aliases for numbered equations.
>
> **Companion skills**: Load `paper-engineering` for the Lean→paper pipeline
> and audit framework. Load `chinese-math-paper` for Chinese translation.
>
> See `references/chinese-paper-formatting.md` for Chinese mathematical paper
> formatting conventions — ctexart document class, Chinese theorem names
> (定理/引理/证明), punctuation rules (use English `.` not Chinese `。` to
> avoid confusion with subscript ₀), and translation workflow.
>
> See `references/F1F2-endpoint-pattern.md` for the Lean-verified F1/F2
> structure for endpoint reduction in derivation-classification papers.\n>\n> See `references/journal-formatting.md` for journal-specific LaTeX\n> formatting requirements — AMS vs Elsevier, font sizes, microtype,\n> elsarticle vs amsart, and a checklist for typesetting quality.


## Core Principle

A mathematical paper is **not** a sequence of formulae. It is an argument
that **communicates an idea** to a human reader. Every sentence should
answer: "Why am I being told this now?"

---

## I. Organization (from Tao & Halmos)

### 1. Logical order, not discovery order
Never present results in the order you discovered them. Organize for
**understanding**, not chronology.

### 2. The proposition-first rule
Every major milestone must be a self-contained Proposition or Theorem,
placed **before** its proof. The reader should know the destination
before the journey. Do NOT delay the punch line to the last page.

### 3. Lemma placement
A lemma should appear **near where it is used**. If used once, put it
right before. If used throughout, put it in a preliminary section.

### 4. Difficulty gradient
- Front of paper: easy, tangible progress → reward the reader
- Back of paper: technical machinery → the reader is now comfortable
  with your notation and can handle more.

### 5. Peripheral material
- Non-essential results → Remarks, footnotes, or Discussion sections
- Necessary but different-flavor (e.g., pure computation) → Appendix

### 6. Section boundaries
- Each major turning point → new section
- Closely related facts → same section
- Each section opens with a brief paragraph: "What this section does and why"

### 7. The spiral method (Halmos)
Write §1, write §2, rewrite §1, rewrite §2, write §3, rewrite §1, rewrite
§2, rewrite §3... Constantly go back and revise earlier sections as later
ones crystallize.

---

## II. Motivation (from Tao)

### 8. The reader must always know WHY
For every step, the reader should know:
- What is the near-term goal? What is the long-term goal?
- How does this step advance toward them?
- Why is this claim plausible (or, if surprising, exactly why it's surprising)?

### 9. Label informal reasoning
Heuristic, intuitive, or motivational reasoning is **welcome** — but must
be clearly labelled as such. Use Remarks or footnotes to distinguish
informal discussion from rigorous proof.

### 10. Toy cases before general results
Before the most general theorem, discuss a concrete special case first.
This is worthwhile even if the toy case is already known. The key is
showing how the toy proof generalizes.

### 11. Section openings
At the start of each section: state the milestone, explain why it matters,
sketch the proof strategy for this section.

---

## III. Language (from Tao & Halmos)

### 12. English over symbols
The same logical statement `P(x) ∧ Q(y)` can be expressed 20+ ways in
English, each carrying different **emphasis, causality, importance, and
context**. English communicates on many more levels than symbols alone.

Use English to signal:
- "Therefore" → logical consequence
- "Moreover" / "Furthermore" → additional fact, same importance
- "In particular" → special case
- "However" / "On the other hand" → contrast
- "Unfortunately" / "Fortunately" → emotional valence for the argument
- "For future reference" → bookmarking a fact for later use
- "Meanwhile" → parallel independent fact
- "Equivalently" → logical equivalence
- "More generally" → generalization

### 13. Halmos' rule: resist symbols
> "The best notation is no notation; whenever it is possible to avoid the
> use of a complicated alphabetic apparatus, avoid it."

If a sentence can be written clearly in words, do not replace it with
`∀∃⇒∧∨` just because you can.

### 14. Notational consistency
Use the same symbols as the previous literature whenever possible.
Expert readers recognize standard notation instantly. Changing it creates
unnecessary cognitive load.

---

## IV. Voice (from Tao)

### 15. Your voice = your interpretation
Do NOT copy paragraphs from prior papers even with attribution. Paraphrase
and reinterpret. Your voice is your **understanding** of the mathematics,
not someone else's prose.

### 16. Standard arguments, standard structure
If an argument is standard in the literature, structure it in the standard
way. Expert readers will recognize it and read faster.

### 17. When duplication is acceptable
Only duplicate existing material when:
- Making a historical point (with proper quotation)
- The original source is obscure or unavailable
- The form of the argument motivates later work in your paper
- The result you need differs slightly from the known version

---

## V. Concrete Writing Rules (from Halmos)

1. **Say something** — have something to say. Empty papers are useless
   no matter how well written.

2. **Speak to someone** — know your audience. Ask: "Who am I trying to reach?"

3. **Organize** — arrange material to minimize reader resistance, maximize insight.

4. **Use consistent notation** — notation deserves careful thought and design.

5. **Write in spirals** — constantly revise earlier sections as later ones develop.

6. **Watch your language** — correct grammar, proper word choice, correct punctuation.

7. **Be honest** — anticipate difficulties and defuse them in advance.

8. **Remove the irrelevant** — delete everything that doesn't serve the
   core argument. "Don't dress up trivialities as theorems."

9. **Use words correctly** — think about the "small words" of logic and
   intuition, and the specialized terms that carry deep mathematical meaning.

10. **Resist symbols** — use words whenever possible over complex notation.

---

## VI. AMS Formatting Conventions

### Theorem environments
Standard hierarchy: Theorem > Lemma > Corollary > Proposition.
Supporting: Definition, Example, Remark, Note, Conjecture, Problem.

Numbering: by section (`\numberwithin{theorem}{section}`) or sequential.

### Citation systems (pick one, use consistently)
1. **Numeric** (most common): [1], [2], ... by order of appearance
2. **Author-year**: (Morrison 2023)
3. **Alphanumeric**: [DL99] = Denef-Loeser 1999

Always include MR (Mathematical Reviews) numbers in references.

### Font conventions
| Element | Font |
|---------|------|
| Function names (sin, lim, det, max) | Roman (upright) |
| Variables (x, y, f(x)) | Italic |
| Number sets (ℝ, ℂ, ℕ, ℤ) | Blackboard bold |
| Vectors | Bold |

### Document classes
`amsart` (journal), `amsproc` (proceedings), `amsbook` (monograph).

### The `\sloppy` trap
NEVER use `\sloppy` — it hides overfull lines by stretching word spacing,
making the page look unprofessional. The correct fix: remove `\sloppy`,
add `\usepackage{microtype}`, and use `\tolerance=500` +
`\emergencystretch=0.5em` for hard lines. Then manually fix the ~2–3 worst
remaining overfulls by rewording. Full details in `references/journal-formatting.md`.

### amsart textwidth note
amsart uses the same text block (360pt) for 10pt and 12pt — switching font
size does not change page density. 12pt is actually better for readability
(~65 chars/line vs ~78 at 10pt). Do NOT reflexively recommend 10pt as a fix
for overfull boxes.

### elsarticle vs amsart — journal class comparison

When the user wants their paper to visually match a published reference (e.g.,
Ou-Wang-Yao 2007 in LAA), compare the document classes. Two-axis analysis:

| | amsart 12pt | elsarticle 3p |
|---|:---:|:---:|
| Paper | US Letter | A4 |
| Font | CM 12pt | Times ~10pt |
| Textwidth | 360pt (5in) | 468pt (6.5in) |
| chars/line | ~60 | ~89 |
| Margins | ~1.76in | ~0.9in |
| Target | AMS journals | Elsevier (LAA, J. Algebra) |

elsarticle 3p produces the look of published LAA papers. Frontmatter uses
`\begin{frontmatter}...\end{frontmatter}` with `\title`, `\author[1]{Name}`,
`\address[1]{Address}`, `\begin{keyword}...\end{keyword}`. The `\runningtitle`
command does NOT exist in elsarticle — running heads are auto-generated.
Theorem numbering and `\DeclareMathOperator` work identically to amsart.
Full migration guide in `references/journal-formatting.md`.

### Visual hierarchy in long proofs (pre-attentive scan)

A proof over 200 lines appears as a "wall of text" not because of content
density but because the reader's **pre-attentive scan** finds too few anchors.
The eye automatically pauses at: Lemma numbers, QED boxes, **bold text**,
indented blocks. Italic text and smallskip spacing do NOT register.

When a proof has 10+ subcases, analyze along TWO independent axes:

1. **Hard structure** (Lemma environments, QED boxes) — tells the reader
   "one logical unit ends, another begins." Target: 1 per ~100 lines.

2. **Soft landmarks** (bold labels, enumerate indentation, displayed equations)
   — gives the scanning eye pause points within a logical unit. Target: 1
   per ~25 lines.

The two axes are complementary, not alternatives. Fixing only one leaves
the other dimension broken.

Technique: **extract intermediate facts as lemmas** (mirrors Ou's pattern
of a separate Lemma 3.1 before the main Theorem 3.2 proof) AND **make
subcase labels bold** (`\textbf{1a.}` not `\textit{Subcase 1a:}`).

When the user says "排版不太美观" or "文字太紧凑", diagnose the visual
hierarchy first — count hard breaks and soft landmarks per page — before
proposing font-size or margin changes.

### Terminology: "trivial" 1/2-derivation

Follow Kaygorodov-Khrypchenko (2023): "multiplication by a field element"
= **trivial** 1/2-derivation. Define in §2 where `\id` is introduced:
`For any s∈F, s\id is again a 1/2-derivation; we call such maps trivial.`
Replace all "scalar multiple of the identity" / "identity scaling s" /
"the scalar s" with "trivial part" / "trivial parameter". The `\id` command
and `s\id` notation stay unchanged.

---

## VII. Anti-Patterns — What Makes a Paper NOT Read Like Mathematics

These are the most common failures when translating Lean proofs to papers:

### A. "Lemma then Proof" without motivation
BAD: A sequence of Lemma-Proof, Lemma-Proof, Lemma-Proof with no text
between them explaining why each lemma exists.

FIX: Before each lemma, one sentence: "To control the endpoint coupling,
we first establish:"

### B. Symbol-only sentences
BAD: `∀i,j with 1≤i<j≤n, coeffOf(D,i,j,i,j) = 0`
FIX: "All diagonal coefficients vanish."

### C. Missing section openings
BAD: Section starts immediately with a Lemma, no context.
FIX: "In this section we prove that the centered part has image
contained in the ideal I. This is the dominant step in the constraint
reduction, collapsing d target channels to 3."

### D. Development-order narrative
BAD: "We first tried X, but that failed because Y. Then we discovered Z..."
FIX: The paper presents the **final** logical argument. The discovery
process belongs in a separate historical note, if anywhere.

### E. Lean names in body text
BAD: "By `coeffOf_cond` we have..."
FIX: "The half-Leibniz condition on the triple (E_ik, E_kj, T) gives..."

### F. Technical machinery before payoff
BAD: 15 pages of technical lemmas, then the main theorem on page 16.
FIX: State the main theorem in §1. State key milestones early. Push
technical proofs to the back.

### G. Substantive results outside formal environments
BAD: A labeled equation like `c_k=0` floating in a subsection without a
Lemma wrapper. The equation IS a mathematical result — it should be stated
as a Lemma/Proposition, not as commentary.
FIX: Wrap in `\\begin{lemma}...\\end{lemma}`. The computation text becomes
its proof. If the same result appears in multiple places (e.g., proved in
a floating computation AND a separate Lemma), merge into one Lemma.

### H. Strategy overview stating unproven results as facts
BAD: \"Step 3: The half-Leibniz condition forces c_k=0.\" — present tense,
asserting a fact before the reader has seen its proof.
FIX: Strategy overviews must use future/reference language: \"Lemma 3.3
will establish c_k=0\" or \"The endpoint reduction shows...\" \nThe rule: if the proof hasn't appeared yet, the strategy can only PREVIEW,
not CLAIM.

Diagnose these with `references/structural-wrapper-audit.md`.

---

## VIII. Pre-Submission Checklist

Before submitting, read the paper as a **hostile referee** and check:

- [ ] Every section opens with a purpose statement
- [ ] Every lemma is preceded by a motivation sentence
- [ ] No sentence is purely symbols when English would be clearer
- [ ] Notation is consistent throughout and matches the literature
- [ ] The main theorem is stated in the introduction
- [ ] Key milestones appear before their proofs
- [ ] Technical details are at the back, easy progress at the front
- [ ] No "we first tried..." — only the final logical argument
- [ ] Lean names appear only in the appendix, never in body text
- [ ] Peripheral results are in remarks/appendices, not the main flow

---

## XI. Notation Rules (from Conrad, Poonen)

### Symbol Inheritance for Derivation-Classification Papers (2026-07-11)

When choosing notation, follow: **Ou-Wang-Yao (2007) → Kaygorodov-Khrypchenko (2023) → keep ours**.
Full convention table and rationale in `references/symbol-inheritance.md`.

### Qualitative Paragraph Reading Framework

To compare prose style against a reference paper, use the Q1-Q6 paragraph analysis:
what it does, what's omitted, what the reader fills, how paragraphs connect,
inevitability, and emotional arc. Full method and worked examples in
`references/qualitative-reading-framework.md`.

## XII. Notation Rules (from Conrad, Poonen)

### Don't start sentences with symbols
Bad: `2 is irrational.` → Good: `The number √2 is irrational.`
Bad: `n = 2m for some m ∈ Z.` → Good: `Thus n = 2m for some m ∈ Z.`

### Separate symbols with words
Bad: `If n≠0 n²>0.` → Good: `If n≠0 then n²>0.`
Bad: `Consider x_k 1≤k≤n.` → Good: `Consider x_k for 1≤k≤n.`

### Never use ∀, ∃, ∧, ∨ in body text
These are for logic papers only. Write "for all", "there exists", "and", "or".

### Never use blackboard abbreviations
No WLOG, s.t., iff in formal text. Write them out.

### One meaning per variable
Bad: `a=2m and b=2m, for some integer m` (reuses m, proves all sums are multiples of 4!)
Good: `a=2m and b=2n, for some integers m and n`

### Define all notation
Bad: `Since n is composite, n=ab.` 
Good: `Since n is composite, n=ab for some integers a,b greater than 1.`

### Notation fits context
Use p for primes, x for set elements, not m for a prime or t for an element of X.

---

## X. Equations, Definitions, and Display (from Conrad, Poonen)

### Display important equations
If you'll refer back to an equation, display it on its own line and label it (2.1), (2.2).

### Equations are sentences — punctuate them
Equations ending a sentence get a period. Equations mid-sentence get a comma if needed.

### Chain equalities vertically
```
(x+1)³ = (x+1)²(x+1)
       = (x²+2x+1)(x+1)
       = x³+3x²+3x+1.
```

### Don't confuse equations with expressions
`x²-3x+4` is an expression. `x²-3x+4 = 0` is an equation.

### Keep theorem statements short
Definitions should precede the theorem. Break long arguments into lemmas even if used once.

### Definitions: term only
Don't put properties or examples inside a Definition environment. They belong after.

### Single theorem numbering
Use one counter for all: Theorem 2.1, Lemma 2.2, Corollary 2.3 — not Theorem 2.1 and Lemma 2.1.

### "So that" ≠ "such that"
"So that" conveys purpose ("in order that"). "Such that" imposes a condition.

---

## XI. Title, Abstract, Introduction (from Poonen, Pak)

### Title
- Long enough to convey content, specific enough to distinguish from other papers
- Omit "A note on" and "Remarks on"
- If you prove "all tennis balls are white", title it exactly that
- If you disprove a conjecture: "Not all tennis balls are white"

### Abstract
- State main results, as dry facts. No personality, no citations.
- Length: 0.3–0.5 lines per page of paper (10-page paper → 3–5 line abstract)
- Self-contained; references to body of paper by section number only if essential

### Introduction
- Hardest section. Write first draft early, rewrite completely after paper is done.
- Get to your first theorem on page 1, or at worst page 2.
- Skip history unless directly relevant to your result. Delegate to Final Remarks.
- Outline paper structure in the last paragraph/subsection.

### Foreword (Pak's invention, for longer papers)
- A nontechnical introduction to your Introduction. 1 page max.
- Literary, big-picture, "why this matters" — the only place to shine stylistically.
- Opposite of Zinsser's advice: your first paragraphs ARE the place to sound like yourself.

### Final Remarks as Endnotes (Pak)
- Expanded footnote section. Each subsection 1 paragraph to 1 page.
- Order by decreasing importance: history/giving credit → future directions → conjectures.
- Cite most related work here, NOT in the body. In the body, only cite results used as lemmas.
- Insert "(see §6.4)" links from the body for interested readers.
- Keep updating on arXiv after publication.

---

## XII. Citations and References (from Poonen, Pak)

### Citation precision
- Include theorem/page/section numbers: [A, §3.1] or [A, Thm 3.2]
- For arXiv preprints: include version number or precise date
- Cite published version over preprint when available
- "Forthcoming work" → only if publicly available preprint exists

### Reference style (Pak's advice)
- Use alphanumeric: [SY09], [Woo96], not pure numeric [1], [2]
- For 5+ authors: [A+13]; same author/year: [Tra15a], [Tra15b]
- Do NOT use BibTeX unless you're an advanced user
- Include arXiv number or free link for unpublished work
- Abbreviate first names, omit book series/issue numbers

### Citation placement
- Introduction: only most directly relevant papers
- Body: only results used as lemmas (with precise location)
- Final Remarks: everything else (history, related work, context)

---

## XIII. Serre's Deep Warnings (from "How to Write Mathematics Badly")

These are presented as things NOT to do. Each is a common failure mode.

### The "a" → "the" switch (indefinite → definite article cheating)
BAD: Theorem 1. There exists **a** map f: X→Y with property P. Theorem 2. **The** map f is continuous.
Why bad: Theorem 1 only says Isom(X,Y)≠∅. "The map f" pretends you've chosen one. If your proof actually constructs f, say so ("the map constructed in the proof of Theorem 1").

**Fix**: Construct the object first (in a lemma or explicit construction), THEN state its properties.

### Commas doing double duty
BAD: "If A, B, C." — comma means "and" between A and B, but "hence" between B and C.
**Fix**: "If A and B are true, so is C." Never let one comma do two different jobs.

### Symbols replacing verbs
BAD: Theorem: Compl. expr. L(χ,s) = eq. compl. expr. (2iπ)^ℓ ∈ Q.
The entire theorem is a formula; "∈" is the verb. Reader must hunt for what's being claimed.
**Fix**: State in words what the formula means.

### Unspecified constants
BAD: "|Af| < C|f|, where C is a constant." Does C depend on f? On A? On other parameters?
**Fix**: Use Vinogradov notation with subscripts: |Af| ≪_A |f|, or state all dependencies explicitly.

### Commutative diagrams claimed, not proved
In homological algebra, it's standard to claim diagrams commute without proof. Serre: "I don't think the calculus of diagrams is on a very solid basis." Know this is a gap.

### The empty proof symbol
Using □ on a lemma with no proof and no explanation. Nowadays, this is the "follows from definition" escape hatch.

### Proof vs. Bourbaki proof
"A proof is accepted by experts. A Bourbaki proof is accepted by non-experts." Aim for the second.

### Identifications are lies we accept
N⊂Z⊂Q⊂R is not literally true in ZFC (elements have different cardinalities). Some identifications are necessary. The art is knowing which.

### Details over ideas, or ideas over details?
Serre's preference: "If you give me a proof with all the details, I can find the ideas behind. If you give me the ideas and not the details, I may get completely stuck." Write detailed proofs.

---

## XIV-bis. Writing Audit Tools

After drafting a proof section, run three audits:

- **Proof completeness audit** (`references/proof-audit-methodology.md`):
  Systematic backward-trace from the final theorem through every reduction
  step. Check for dangling references, internal contradictions between
  reduction claims and the parameter tally, and unsubstantiated coupling
  claims. This audit catches the most dangerous gaps — claims that a
  variable is zero when the parameter tally lists it as a survivor.

- **Sentence structure metrics** (`references/sentence-structure-metrics.md`):
  8 quantifiable targets for math/meta ratio, subject variety, length distribution,
  and clause nesting depth. Use to verify the section reads like Ou's §3.

- **Negative space checklist** (`references/negative-space-checklist.md`):
  10 categories of content that Ou never writes. After drafting, delete every
  sentence matching these patterns. The editing test: remove each sentence;
  if the argument still works, the sentence was unnecessary.

- **Proof pitfalls from v3 audit** (`references/proof-pitfalls-v3.md`):
  Common proof errors discovered during cross-reference with Lean
  formalization: chain bridge on wrong source, strategy summary bounds
  mismatch, unverified adjectives, corollary-as-tautology, forward
  references in decomposition, and the i.e. allowed/forbidden distinction.

- **Forward-reference audit** (`references/forward-reference-audit.md`):
  Systematic trace of every symbol through the paper in reading order.
  Catches symbols used before they are defined — the most common being
  `\\id` used in §1 decomposition lemmas before its §2 definition. This
  is a distinct audit from the structural ones above; run it last, once
  all sections are in final order.

- **Structural wrapper audit** (`references/structural-wrapper-audit.md`):
  Checks that every substantive mathematical statement has a proper formal
  wrapper. Three checks: (1) every labeled equation is inside a Lemma or
  proof, (2) strategy overviews use future/reference tense, not present-fact
  tense for unproven results, (3) recurring facts used as premises have
  named environments, not bare `\\textit{Observation.}` labels. Run BEFORE
  the forward-reference audit — wrappers must be in place before you trace
  where symbols are defined.

- **Per-file audit template** (`references/per-file-audit-template.md`):
  Systematic methodology with A-group (grep scans), B-group (per-sentence),
  and C-group (per-paragraph deletion tests). Includes mandatory A-group
  re-scan after all fixes. Catches bugs like bounds mismatches, false
  "maximal" claims, and "i.e." that survive initial scans.

### Don't Coin Names for Numbered Equations (2026-07-11)

If a concept is already expressed as a numbered equation (e.g., `\eqref{eq:halfder}`),
do NOT create a separate English name for it and use it throughout the paper.
This creates redundant terminology that:

- Adds jargon the reader must learn
- Is not used by reference papers (Ou, KK)
- Is literally just "the equation you already numbered"

**Bad**: "applying the half-Leibniz condition to the pair..."
**Good**: "by \eqref{eq:halfder} applied to the pair..."

The numbered equation IS the name. 12 occurrences of "half-Leibniz condition"
were replaced with `\\eqref{eq:halfder}` — zero information loss, cleaner prose.

### Don't Name Proof Techniques (2026-07-11)

The broader pattern: beyond equations, do NOT give formal names to proof
steps, sub-lemmas, or computational techniques. Reference papers use
**zero** coined technique names — every reference is by lemma or equation
number. Subsection titles are purely descriptive (\"Upper triangular matrix
algebra\"), not named after a technique (\"The Chain Bridge Lemma\").

Source-code comparison with KK 2023 confirms this: their LaTeX source
has 0 named techniques. See `references/no-coined-technique-names.md`
for the full analysis.

In our v3 paper, the affected coinages are: \"chain bridge\", \"width
reduction\", \"diagonal propagation\", \"image restriction\", \"endpoint
reduction\", \"a/c-channel\", and \"the Observation\".

**Fix pattern**:
- Subsection titles → descriptive (\"Reduction to adjacent sources\" not
  \"Width reduction\")
- Inline references → lemma number (\"by Lemma 3.1\" not \"by image restriction\")
- Metaphors → direct description (\"coefficient of E_{1,n-1}\" not \"a-channel\")
- Capitalized facts → plain language (\"note that\" not \"the Observation\")

### Pre-Attentive Scan

When a proof section feels visually oppressive, count the **pre-attentive anchors**
per page — elements the eye registers before reading:

| Anchor Type | Example |
|-------------|---------|
| Lemma/Theorem number + QED box | Hard structural break |
| Bold text | Case labels, section heads |
| Indented blocks | Enumerate, itemize |
| Displayed equations | White space interruption |

A healthy math paper has **3-4 anchors per page**. A proof section with only
1-2 anchors (one Lemma + one QED across 3 pages) will feel like a "wall."
Fix by:
- Promoting inline claims to numbered Lemmas
- Making subcase labels **bold** instead of `\textit`
- Using displayed equations to break text flow

### Bracket-Expansion Compression

When a proof contains bracket expansions like:
> "only the term E_{ab} can produce the target, via [E_{ab},E_{cd}]=E_{uv}, contributing π..."

Replace with declarative:
> "the term E_{ab} contributes π..."

The reader knows how bracket expansions work. Showing the intermediate computation
reads like lecture notes, not a published paper. This technique removed 85 lines
(46%) from one proof section with zero mathematical content loss.

### Downshift your style
- Your readers include non-native English speakers with short attention spans
- Ten "therefore"s are fine if they're clear. Don't vary style for elegance.
- Short sentences. Present tense. Standard American spelling ("coloring" not "colouring").
- Repetition of notation reminders is GOOD, not redundant.

### Ignore all rules for clarity
When any style rule makes the math less clear, break it. (But try rewording first.)

### LaTeX macros for everything
`\al` for `\alpha`, `\rT` for `\textrm{T}`, etc. Makes global notation changes trivial.

### Figures: many but small
3–5 cm. Human eye is good. Use Ctrl+ if needed. Align multiple copies with different detail levels.

### You're never done
Let the paper stew a week. Email colleagues. Update arXiv even after publication. Blog about it. Give talks and link the slides.

---

## IX-bis. Sources Collected (2026-07-08 Update)

The following four sources were downloaded and fully extracted during this session.
Their principles are now synthesized into the sections above.

1. **Keith Conrad** — "Advice on Mathematical Writing" (10 pp, 22,834 chars)
   - Notation rules, equation display, definitions, font conventions, LaTeX tips
   - URL: `https://kconrad.math.uconn.edu/blurbs/proofs/writingtips.pdf`

2. **Bjorn Poonen** — "Practical Suggestions for Mathematical Writing" (6 pp, 13,964 chars)
   - 65 concrete rules: justification, title/abstract/intro, LaTeX, nitpicks
   - URL: `https://math.mit.edu/~poonen/papers/writing.pdf`

3. **Jean-Pierre Serre** — "How to Write Mathematics Badly" transcript (16 pp, 34,606 chars)
   - Ironic lecture: indefinite→definite article, commas, constants, diagrams, identifications
   - Transcript URL: `https://bpb-us-w2.wpmucdn.com/web.sas.upenn.edu/dist/0/713/files/2025/05/How_to_write_mathematics_badly__transcript_.pdf`
   - Video: `https://www.youtube.com/watch?v=ECQyFzzBHlo`

4. **Igor Pak** — "How to Write a Clear Math Paper: 21st Century Tips" (18 pp, 58,700 chars)
   - Modern perspective: competitive clarity, Final Remarks as endnotes, reference hierarchy, downshift style