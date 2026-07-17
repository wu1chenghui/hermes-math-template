# Component Comparison Framework — Whole-Paper Ou Style Matching

> Discovered 2026-07-11 during D1-D6 analysis of Ou 2007 vs our HalfDer(N_n) v3.
> Use when checking whether a paper matches Ou's style across ALL sections,
> not just the main proof.

## The 13-Component Checklist

| # | Component | Ou's Pattern | What to Check |
|---|-----------|-------------|---------------|
| 1 | Title | Declarative, no "On"/"Remarks on" prefix | Match |
| 2 | Abstract | 3 sentences: object → method → result. No citations | Match |
| 3 | Literature | 2-4 citations, 2 sentences max | Match |
| 4 | Structure paragraph | "This paper is organized as follows. In Section 2... In Section 3..." — standard, not a tell | Match |
| 5 | Preliminaries | Embedded at end of Introduction (~25 lines). "Before giving the main result, we introduce..." | Only merge if <50 lines total |
| 6 | Construction format | (A)-(E) letter labels, one per construction. "Then it is easy to check" — never expand verification | Use if ≤5 equal-rank constructions |
| 7 | Construction verification | Never expands computation. Uses "by (1.1)", "by index constraints" | Compress for standard constructions; expand for novel ones |
| 8 | Proof architecture | Numbered Steps. Each shrinks problem: φ → φ₁ → φ₂ → φ₃ → φ₄. Each Step heading = goal. | Adopt if proof has progressive reduction chain |
| 9 | Reduction language | "Now we denote φ − X by φ₁" — name the reduced object | Optional; use if proof has 3+ reduction steps |
| 10 | Appendix | None. All content in 3 main sections | Only add if proof would bloat beyond 1:2 setup:proof ratio |
| 11 | Acknowledgments | "The authors thank the referee for his helpful suggestion." + funding | Match |
| 12 | References | Numeric [1]-[N]. No equation numbering for refs | Match |
| 13 | Section count | Exactly 3 numbered sections + unnumbered Introduction | Match when possible; separate appendix is acceptable |

## Key Principle: Differences Need Justification

Every deviation from Ou's pattern must have a mathematical reason, not a stylistic preference:

| Deviation | Justification Required |
|-----------|----------------------|
| Separate §1 Preliminaries | Preliminaries too long to embed in Intro (>50 lines) |
| More than 5 constructions | Cannot collapse into (A)-(E) without losing clarity |
| Expanded construction verification | Construction is novel, not a standard type |
| Appendix exists | Proof has extensive bracket expansions that would break 1:2 ratio |
| More than 3 sections | Proof complexity requires additional structure |

If no mathematical justification exists for a deviation, align with Ou.
