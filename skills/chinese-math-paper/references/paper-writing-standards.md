# Mathematical Paper Writing Standards

> Established through systematic comparison with reference papers in the
> nilpotent Lie algebra / half-derivation literature (2026-07-11).

## Reference Papers Analyzed

| Paper | Year | Topic | Style |
|-------|------|-------|-------|
| Kaygorodov-Khrypchenko | 2023 | transposed Poisson on T_n(F) | lean, no named techniques |
| Ghimire-Huang | 2016 | derivations of block upper triangular | verbose (50 "we"), section-based eq numbers |
| Yusupov-Madrakhimov-Vaisova | 2025 | 1/2-derivations on quasi-filiform Leibniz | direct, 22 "we" |
| Ou-Wang-Yao | 2007 | derivations of N(n,F) | 6 pages, very concise |

## KK-Style Standards (Target)

### What a math paper should NOT have

1. **Named proof techniques** — "chain bridge", "width reduction", "diagonal propagation",
   "image restriction", "endpoint reduction". Use equation numbers and lemma numbers instead.
   KK: 0 named techniques. Our v1: 7+.

2. **Numbered step frameworks** — "Step 1/2/3/4". Use natural prose transitions
   ("First, we prove...", "To prove..., consider...").

3. **Meta-commentary about proof structure** — "analyzed through the following reductions".
   Let the mathematics convey structure.

4. **Metaphors as terminology** — "a-channel", "c-channel", "source", "target".
   Use direct index descriptions: "coefficient of E_{1,n-1}", "(i,j)", "at (u,v)".

5. **Tutorial voice markers** — "Observe that" as a standalone named paragraph,
   "We now prove" as a section opener, "Recall that", "Note that".
   KK uses "Observe that" 6x but inline, never as a labeled block.

6. **Ornamental content** — Dynkin involution symmetry listing (unused in proofs).
   If a definition isn't used, don't include it.

7. **Over-granular subsections** — one-equation subsections, "Parameter tally" as a named
   subsection. Let lemmas flow naturally within broader sections.

8. **Proof method in abstract** — KK, GH, Yusupov all use pure-results abstracts.
   "The upper bound is obtained by..." → "dim = n+5, and we construct an explicit basis."

### What a math paper SHOULD have

- Direct, declarative prose
- Lemmas flowing in sequence without subsection-per-lemma
- Pure-results abstract
- Equation references by number, not by name
- Natural prose transitions between proof segments
- No discovery narrative, no pedagogical motivation, no defensive language

### Style Metrics (from comparison)

| Metric | KK | GH | Yusupov | Our v3 |
|--------|:--:|:--:|:--:|:--:|
| "we" count | 10 | 50 | 22 | 5 |
| "Thus" | 5 | 1 | 0 | 25 |
| "Hence" | 4 | 1 | 0 | 0 |
| "Therefore" | 0 | 14 | 1 | 0 |
| Named techniques | 0 | 0 | 0 | 0 ✓ |
| Formal Definitions | 0 | 2 | — | 0 |
| "Step" labels | 0 | 0 | 0 | 0 ✓ |
| Abstract style | results | results | results | results ✓ |

### Debranding Methodology

When auditing a paper for self-coined terminology:

1. Search for ALL capitalized/emphasized concept names in the paper
2. Cross-reference with reference papers (grep their LaTeX source for same terms)
3. Classify each term: standard / descriptive / coined
4. For coined terms, replace with:
   - Equation reference if it's a named identity
   - Lemma reference if it's a named lemma
   - Direct index/coordinate description if it's a metaphor
   - Natural prose if it's a structural label
5. Recompile and verify no residual references

## Paper-Lean Verification Methodology

When cross-verifying a paper against Lean formalization:

1. Map each paper theorem to its Lean equivalent (use `lean_file_outline`)
2. For each theorem, compare: conditions, conclusion, proof approach
3. Check that all edge cases (char conditions, index bounds) match
4. Verify that no theorem in Lean is missing from the paper, and vice versa
5. Flag discrepancies in: assumptions, covered ranges, proof logic
