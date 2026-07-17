# Compression Workflow for English Math Papers

> Captured 2026-07-12 from 5-round compression of 741→610 lines.
> Reference papers: Ou-Wang-Yao (2007, LAA), KK (2023, arXiv:2305.00727).

## Core Principle

Compression is NOT about formatting (display vs inline). It's about
removing content the reader can infer. Reference papers say "it is easy
to check" — we should not expand that check.

## The 5-Round Pattern

Each round follows the same structure:

1. **Read entire paper** with two lenses: "what would Ou delete?" and
   "what would a reader already know?"
2. **Identify specific blocks** — concrete line ranges — never vague
   impressions
3. **Draft exact replacement text**
4. **Verify against Lean formalization** — every change must be
   mathematically equivalent
5. **Execute and compile** — 0 errors required before next round

## Classified Redundancy Patterns

### Pattern A: Display equations for single-step algebra
**Signal**: `\[...\]` containing one equation line with simple algebra.
**Fix**: Inline. Example: `π_{1,i+1}=2X` as `$...$`, not display.
**Total savings**: ~20 lines across 3 blocks.

### Pattern B: Bracket expansion verification
**Signal**: Showing all 3 terms when 2 are zero. Phrases like
"the bracket-left term: ... Since A=0, B=0, and C≠0, we get..."
**Fix**: "the bracket-left term contributes X (only component Z
interacts non-trivially)."
**Total savings**: ~12 lines across F1/F2 proofs.

### Pattern C: Signposting / meta-commentary
**Signal**: "First, we prove...", "We distinguish three cases...",
"Five additional derivations are supported at..."
**Fix**: Delete. The structure speaks for itself.
**Total savings**: ~8 lines.

### Pattern D: Verbose commuting-pair justification
**Signal**: "commutes because the index sets {k,k+1} and {1,2} are
disjoint. By (1), [φ(x),y]+[x,φ(y)]=0, projected to (u,v), expands
as follows."
**Fix**: "the commuting pair (X,Y) gives Z." One line.
**Total savings**: ~25 lines across Lemma 3.2 cases.

### Pattern E: Redundant condition restatement
**Signal**: Repeating global conditions (n≥5, char≠2,3) inside proofs.
**Fix**: Delete. Global condition declared once after Theorem 1.1.
**Total savings**: ~5 lines.

### Pattern F: Descriptive sentences about constructions
**Signal**: "this is the image of ω₁ under the Dynkin involution..."
**Fix**: Delete. Construction motivation is not mathematics.
**Total savings**: ~5 lines.

### Pattern G: Unused properties
**Signal**: "This ideal is abelian and satisfies..." — "abelian" never
used in any proof.
**Fix**: Delete unused adjective.
**Total savings**: ~2 lines.

## Construction Verification Density

Ou's approach: define construction → "it is easy to check" → move on.
Our approach before compression: define → verify each case → conclude.

After compression, we kept verification for non-trivial constructions
(ω₂ with factor ½, ω₃ with factor 2) but compressed trivial ones.

Rule of thumb: if the verification shows WHY a specific coefficient
(½, 2, -1) is needed, keep it. If the verification is just bracket
algebra, compress to one line.

## Lean Verification Protocol

Every compression change must pass:
1. Identify the corresponding Lean theorem/lemma
2. Confirm the compressed claim is exactly what Lean proves
3. Check index ranges, conditions, and edge cases match

## Page Count vs Content Density

| Stage | Lines | Pages | PDF |
|-------|:---:|:---:|:---:|
| Initial | 741 | 6 | 75 KB |
| Round 1 | 691 | 6 | 74 KB |
| Round 2 | 653 | 5 | 71 KB |
| Round 3 | 635 | 5 | 70 KB |
| Round 4 | 623 | 5 | 69 KB |
| Round 5 | 610 | 5 | 68 KB |

Total: -131 lines (-17.7%), -7.2 KB, -1 page.

## Translation Plan Synchronization

After compression, the translation plan MUST be rewritten. The old plan
referenced deleted content (chain bridge names, Dynkin paragraph,
n=4 exception, old sub-case labels). Key update points:
- Line counts for every section
- Section/subsection titles (many renamed)
- Lemma numbering (verify \ref targets)
- Terminology table (remove deleted terms)
