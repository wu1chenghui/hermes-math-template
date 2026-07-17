# Mathematical Writing Style — Reference Paper Comparison

> Collected 2026-07-11 during paper v3 editing session.
> Source papers: Ou-Wang-Yao (2007), KK (2023), Ghimire-Huang (2016), Yusupov (2025).

---

## Style Metrics Across Reference Papers

| Feature | Ou (6pp) | KK (30pp) | GH (13pp) | Yusupov (10pp) | Our v3 (8pp) |
|---------|:--:|:--:|:--:|:--:|:--:|
| "we" | 44 | 10 | 50 | 22 | 5 |
| "Thus" | 5 | 5 | 1 | 0 | 25 |
| "obtain/gives" | 1/0 | 3/1 | — | 1/1 | 6/18 |
| Informal language | yes (easy/check/get/see) | minimal | no | no | no |
| Named technique subsections | no | no | no | no | removed |
| Case labels | Step 1/2/3 | Case 1/2 | (1)(2) | none | removed |
| Abstract style | pure results | pure results | pure results | pure results | pure results |

## Key Qualitative Difference: Narrative vs Checklist

**Ou writes as narrative**: varied sentence lengths (5-23 words), flowing transitions
("Now we consider...", "Similar as above...", "By this we may..."). The proof tells
a story of reduction.

**Our original wrote as checklist**: uniform short sentences (6-11 words), repetitive
"Thus... Thus... Thus...", subcase labels reading like an outline. The proof was a
verification tree.

## Case Analysis Presentation

When a proof inherently requires case analysis (like our Lemma 3.2 with 10 subcases),
the structure can be improved by:
1. Grouping cases that share the same commuting partner
2. Narrating the common proof once with variations noted inline
3. Avoiding subcase numbering (1a, 1b, 1c, ...) in favor of prose conditions

**Lean-validated merging** (2026-07-11):
- Case 1 (v<n): 1a+1b+1c share E_{v,v+1} partner → merge to "u ≥ i" paragraph
- Case 2 (v=n): (ii)+(iv) share (E_{1,u},E_{i,i+1}) partner → merge to one paragraph
- Result: 10 subcases → 7 paragraphs

## What NOT to Do (confirmed by all reference papers)

- Don't name proof techniques (chain bridge, width reduction, etc.)
- Don't use numbered Step lists in proof overviews
- Don't create subsections for single equations
- Don't use metaphors as technical terms (source, target, channel)
- Don't include ornamental symmetry observations not used in proofs
- Don't split a counting argument into its own named subsection
- Don't describe proof method in abstract

## What IS Normal (present in reference papers)

- Classification names for constructed maps (Ou's "Inner/Central/Extremal",
  our "Trivial/Simple-root/Boundary")
- Informal verification language ("it is easy to check", "it is not difficult to verify")
- "Recall that..." and "Observe that..." (KK uses "Observe" 6 times)
- Short papers with minimal references (Ou: 6 pages, 7 refs)
