# Round 5: Signpost Eradication — Methodology & Results (2026-07-12)

## Context

After R1-R4 reduced the paper from 741 to 623 lines (−16%, 6→5 pages), a
fifth pass targeted the last traces of lecture-note scaffolding: sequencing
words, redundant condition restatements, descriptive framing sentences, and
verbose verification that follows an already-established pattern.

## Targets

### Sequencing signposts
- "**First,** we prove d_k=d_{k+2}." → DELETE "First,"

### Redundant global conditions
- "To prove d₁=d₂ **(this requires n≥5)**, consider (1,5)." → DELETE
  parenthetical (n≥5 is declared globally after Theorem 1.1; Lean's
  `all_diag_equal` hypothesis `hn5` confirms it's globally required)

### Descriptive framing sentences
- "Five additional 1/2-derivations are supported at the Dynkin endpoints."
  → DELETE (the list below speaks for itself)

### Over-explained cancellations
- ω₃'s 3-line commuting-pair verification compressed to 2 lines:
  "Commuting pairs cancel: im(ω₃)⊆I by (3), and the explicit coefficients
  force pairwise cancellation."

### Redundant elimination details
- 5 individual elimination lines → 2-line list:
  "Evaluating at E₁₂→(2,n), E₁₃→(1,n), E_{n-2,n-1}→(1,n-1),
  E_{n-1,n}→(1,n-1), and E₁₂→(1,n-1) eliminates d₁,…,d₅."

### Case signposting
- "We distinguish three cases for u." → DELETE

### Redundant source attribution
- "The family {...} constructed in Section~2 is linearly independent by
  Proposition~1.2. Thus dim = n+5." → "Proposition~1.2 gives the lower
  bound; hence equality."

## Lean Verification

Each change verified against Lean:
- n≥5: Lean's `all_diag_equal` hypothesis `hn5` confirms global requirement
- τ_k: only basis element with non-zero (1,n)-entry at E_{k,k+1}
- ω_i eliminations: each has unique non-zero target at specified pair
- ω₃ cancellation: im(ω₃)⊆I by construction, coefficients (2,1) force pairwise zero

## Result

623 → 610 lines (−13), PDF 68.83 → 67.97 KB. Still 5 pages.

## Updated Cumulative Results

| Round | Lines | PDF Size | Pages | Focus |
|-------|-------|----------|-------|-------|
| Start | 741 | 75.20 KB | 6 | — |
| R1 | 691 | 73.68 KB | 6 | Display→inline |
| R2 | 653 | 70.88 KB | 5 | Prose compression |
| R3 | 635 | 69.73 KB | 5 | Polish + typos |
| R4 | 623 | 68.83 KB | 5 | Micro-polish |
| R5 | 610 | 67.97 KB | 5 | Signpost eradication |

Total: −131 lines (−17.7%), −7.23 KB (−9.6%), −1 page.
All 5 rounds verified against Lean with 0 errors.
