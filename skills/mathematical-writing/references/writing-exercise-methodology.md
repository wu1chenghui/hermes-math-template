# Writing Exercise Methodology

## When to use
When rewriting a paper to match a reference author's style (e.g., Ou-Wang-Yao
or Kaygorodov-Khrypchenko). This is an iterative write→audit→fix→re-audit loop.

## Phase 1: Deep analysis (before writing)

**CRITICAL — Depth spiral**: Do not stop after one round of analysis. The user's
standard is "reads like the reference author wrote it themselves." Achieving
this requires at least 4 rounds of deepening:

| Round | Analysis Type | Deliverable | Duration |
|-------|--------------|-------------|----------|
| 1 | Structure & notation | Section map, notation audit | 30 min |
| 2 | Sentence-level patterns | Sentence opener distribution, length stats, clause depth | 1 hr |
| 3 | Micro-details | Equation template extraction, forbidden-word census, transition vocabulary | 1.5 hr |
| 4 | Narrative arc + decision reconstruction | "Why did they write X instead of Y?" for every paragraph | 2 hr |
| 5 | Negative space | "What would a less experienced author write here?" — catalog omissions | 1 hr |
| 6 | Voice fingerprint | Quantify: hedging rate, math density, let:we ratio, paragraph atomicity | 1 hr |

**When the user says "this is too shallow"**: go deeper, not wider. Don't add
more sections to the analysis — add more *layers* within the same section.
The correct response is: "Here's what I missed. Let me look closer at [next
layer down]."

1. **Forensic sentence reconstruction**: Take one complete proof step from the
   reference paper. Annotate every sentence with the author's decision: what they
   wrote vs what they could have written. Extract the underlying principles.
   
2. **Negative space analysis**: For every paragraph, ask "what would a less
   experienced author write here that the reference author chose not to write?"
   Catalog the omitted content.

3. **Quantitative fingerprinting**: Run automated analysis on the reference paper's
   proof section (§3) to extract:
   - Math vs meta sentence ratio (target: ≥63% math, ≤3% pure navigation)
   - Grammar subject distribution (target: ≥5 types, max ≤15% for any single type)
   - Sentence length distribution (target: 41% short, 34% medium, 25% long)
   - Clause nesting depth (target: avg 3.7, alternating deep→shallow)

## Phase 2: Writing exercises

Write one section at a time. Each exercise is a self-contained `.tex` file.

Rules during writing:
- Use the reference author's sentence templates (not your own defaults)
- Follow the 6-step computation template for bracket calculations
- Follow the (A)-(E) construction template for definition sections
- Apply the pre-flight checklist BEFORE writing each sentence

## Phase 3: Audit

After each exercise, run automated checks:
1. Forbidden words scan (Hence, Therefore, vanishes, contradiction, induction, etc.)
2. Sentence opener variety count
3. Consecutive same-opener detection
4. Forbidden pattern scan (gap narrative, self-praise, "In other words", etc.)

## Phase 4: Fix and re-audit

- Fix all audit failures
- Re-run audit until clean
- Analyze error patterns: which mistakes recur? Why?
- Update the pre-flight checklist with new error patterns discovered

## Phase 5: Assembly

When all exercises are clean:
- Combine into a single `main.tex`
- Verify cross-references between sections
- Final read-through against the reference paper's voice
