# Paper Compression Methodology

> Developed during the v3 paper compression (2026-07-11/12, 741→610 lines, 5 rounds).

## Reference Papers (Density Benchmarks)

| Paper | Pages | Format | Font | Display Eq. | Style |
|-------|:-----:|--------|:----:|:-----------:|-------|
| Ou-Wang-Yao (2007) | 6 | single-column | 9pt | 0 in main proof | "it is easy to check" |
| KK (2023) | 12 | arXiv preprint | 10pt | 5-10 | shows only key equations |
| Our v3 (final) | 5 | elsarticle 3p | 10pt | ~25 | inline where possible |

## Compression Categories

### 1. Display Math → Inline
Single-line equations that don't need visual prominence.
- **Before**: `\[ \pi_{1,i+1} = 2X. \tag{1} \]`
- **After**: `$\pi_{1,i+1}=2X$`

### 2. Proof Signpost Removal
- "First, we prove X" → "We prove X"
- "We distinguish three cases" → just list the cases
- "We split according to whether..." → `\textbf{Case 1: ...}`

### 3. Redundant Restatements
- Don't repeat lemma conditions in the proof
- Don't restate d₁=d₃=... when d_k=d_{k+2} already says it
- Don't remind the reader global conditions (char≠2,3, n≥5)

### 4. Construction Verification Compression
Ou's approach: define construction, add "it is easy to check", move on.
- Our τ_k was 9 lines → 5 lines
- Our ω₂ was 10 lines → 5 lines
- Our ω₄ had a Dynkin motivation paragraph → deleted

### 5. Bracket-Expansion Removal
- `[aE+bE+cE, E] = a[E,E] + b[E,E] + c[E,E]` — show only the non-zero term
- "the generators of I have no index overlap" → "its generators are disjoint"

## Verification Protocol

Every compression must pass:
1. **Lean cross-check**: Does Lean's proof use the same reasoning?
2. **Reader fill-in**: Can a specialist fill in the missing steps mentally?
3. **Reference comparison**: Would Ou or KK include this detail?

## Round-by-Round Audit

| Round | Lines | PDF | Pages | What |
|:-----:|:-----:|:---:|:-----:|------|
| Start | 741 | 75.20 KB | 6 | Original |
| 1 | 691 | 73.68 KB | 6 | d₁=d₂ expansion, Lemma 3.1 u=1, Lemma 3.2(iv) |
| 2 | 653 | 70.88 KB | 5 | §2 constructions, Case 1/2 compressions, F2 bracket |
| 3 | 635 | 69.73 KB | 5 | Minor polish: deleted signposts, typos, redundancy |
| 4 | 623 | 68.83 KB | 5 | "abelian", parity line, ω₂, Case 1 header, bracket-right |
| 5 | 610 | 67.97 KB | 5 | "First", "(n≥5)", Dynkin description, τ_k/ω_i elimination |
