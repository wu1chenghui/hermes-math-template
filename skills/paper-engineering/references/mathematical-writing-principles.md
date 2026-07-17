# Mathematical Writing Principles

> Collected from authoritative sources on mathematical writing.
> Load this when the paper reads like a Lean transcript rather than a math paper.

---

## Source 1: Terry Tao's Writing Advice (4 articles)

### Organise the Paper

1. **Logical layout, not discovery order.** Stream-of-consciousness (results in order discovered) is bad.
2. **Stratify by importance.** Peripheral results → remarks/footnotes. Different-flavor but necessary → appendix.
3. **Section at each turning point.** Related facts in one section.
4. **Propositions BEFORE proofs.** State each milestone as a self-contained Theorem/Proposition BEFORE its proof. Never delay the punch line to the last page.
5. **Lemma near usage.** Lemma X should be near where it's used.
6. **Difficulty gradient.** Technical material → back of paper. Front: easy, tangible progress.
7. **Dependency DAG for large papers.** Boxes = lemmas, arrows = deductions. Group into sections from the DAG.

### Motivate the Paper

1. **Not just formulae.** Reader must know near-term AND long-term objectives at all times.
2. **Label informal reasoning.** Heuristic/motivational reasoning is welcome but MUST be in remarks/footnotes, clearly distinguished from rigorous proof.
3. **Section opening paragraph.** State the milestone, why it matters, proof sketch.
4. **Toy case first.** Before the most general result, discuss a concrete special case — even if known.

### Take Advantage of English

1. **Don't write purely in symbols.** The same logical statement P(x)∧Q(y) has 20+ English realizations, each conveying different emphasis, causality, and importance.
2. **Use English to add dimensions.** "P(x) is true. Furthermore, Q(y) is true." ≠ "P(x) is true. However, Q(y) is true." ≠ "Since P(x) is true, Q(y) is true."

### Write in Your Own Voice

1. **Never copy paragraphs verbatim** (even with attribution). Always paraphrase and reinterpret.
2. **Maintain notational consistency** with prior literature.
3. **Standard arguments → standard structure.** Helps expert readers.
4. **"Own voice" = your interpretation of the mathematics**, not someone else's prose.

---

## Source 2: Paul Halmos, "How to Write Mathematics" (1970)

### Core Philosophy

> "The basic problem is communicating an idea. To do so, and to do it clearly, you must have something to say, you must have someone to say it to, you must organize what you want to say, you must write, rewrite, and rewrite again several times, and you must be willing to work hard on the mechanical details of wording, notation, and punctuation. That is all there is."

### Ten Rules

1. **Say something.** Having something to say is the most important thing. Empty papers are useless no matter how well written.
2. **Speak to someone.** Know your audience. Ask: "Who do I want to reach?"
3. **Organize.** Arrange material to minimize reader resistance and maximize insight. Order by UNDERSTANDING, not discovery.
4. **Use consistent notation.** Letters and symbols deserve careful design. Be uniform and coherent.
5. **Write in spirals.** Write §1, write §2, rewrite §1, rewrite §2, write §3, rewrite §1, rewrite §2, rewrite §3... Continuously revise earlier sections.
6. **Watch your language.** Correct grammar, appropriate word choice, correct punctuation, common sense.
7. **Be honest.** Pave the way for the reader. Anticipate difficulties and resolve them in advance. Pursue clarity, not pedantry.
8. **Remove the irrelevant.** Irrelevant hypotheses, wrong emphasis, or even missing right emphasis can destroy a paper. Delete everything that doesn't serve the core argument.
9. **Use words correctly.** Think carefully about "small words" and mathematical terminology that have profound effects on meaning.
10. **Resist symbols.** "The best notation is no notation; whenever possible, avoid complicated symbolic apparatus." Use words when they suffice.

### Key Aphorisms

- "Don't dress trivialities up as theorems."
- Use □ (tombstone/Halmos) to mark end of proof.
- Writing is a cyclic process of continuous improvement.

---

## Source 3: AMS Style Guide (Journals)

### Document Structure

Standard AMS article uses: Title → Abstract → Sections → Appendix → References.

### Theorem Environments (Enunciations)

Standard hierarchy:
- **Theorem, Lemma, Corollary, Proposition** — numbered, proven results
- **Definition, Example, Remark, Note** — unnumbered or separately numbered
- **Conjecture, Problem, Question** — open

Numbering: by section (`\numberwithin{theorem}{section}`) or sequential.

### Citation Systems (pick one, be consistent)

1. **Numeric (most common):** [1], [2], [3]... by order of appearance. Multiple: [1,3,5] or [4–7].
2. **Author-year:** (Morrison 2023).
3. **Alphanumeric abbreviation:** [DL99] = Denef and Loeser 1999.

**Must include MR number** (MathSciNet Mathematical Review) — permanent identifier.

### Typography

| Element | Style |
|---------|-------|
| Function names (sin, cos, lim, log, max, det) | Roman (upright) |
| Variables (x, y, f(x)) | Italic |
| Vectors or special structures | Bold |
| Number sets (ℝ, ℂ, ℕ, ℤ, ℚ) | Blackboard bold |

### Document Classes

- `amsart` — AMS journal articles
- `amsproc` — conference proceedings
- `amsbook` — monographs

### Core Principle

**Less is more.** Avoid unnecessary words. Keep mathematical notation consistent. Precision above all.

---

## Source 4: Serre, "How to Write Mathematics Badly" (Harvard, 2003)

> NOTE: This talk uses IRONY — Serre describes BAD practices to illustrate GOOD ones.
> Research incomplete (agent timed out). Key points from secondary sources:

Known principles (to be verified against transcript):

- Don't use unnecessary notation or terminology
- Define symbols before using them
- Don't introduce notation you only use once
- Be precise about what you're proving
- Don't over-explain trivial steps; don't under-explain non-trivial ones
- The reader's time is more valuable than the author's

---

## Synthesis: Common Themes Across All Sources

1. **Reader-first.** Every decision — notation, structure, detail level — serves the reader.
2. **Words over symbols.** Use English. Notation is a tool, not the product.
3. **State before proving.** Tell the reader what you're going to prove before proving it.
4. **Structure for understanding, not discovery.** The paper's order ≠ the research order.
5. **Delete ruthlessly.** Every sentence must earn its place.
6. **Rewrite repeatedly.** The first draft is never good enough.
7. **Be precise but not pedantic.** Distinguish rigorous from heuristic. Label each.
8. **Consistent notation.** One symbol = one meaning throughout the paper.
