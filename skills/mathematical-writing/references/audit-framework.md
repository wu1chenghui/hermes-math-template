# Six-Layer Paper Audit Framework

Design: one-pass per-file, every sentence checked. Avoid ad-hoc re-reading.

## Layer 0: Mathematical correctness [must pass first]
- Every theorem/lemma cross-referenced against Lean formalization
- `lean_verify` on key theorems

## Layer 1: Forbidden words (A-group — grep scan)
14 patterns: Hence|Therefore|Consequently|Namely|i.e.|vanishes|contradiction|induction|clearly|obviously|key|crucial|important|fundamental|Moreover|Furthermore|Also|Next|Finally|respectively|immediately|Note that|as follows|belongs to|lies in

## Layer 2: Sentence structure (B-group — per-sentence reading)
Per sentence: grammar subject type, meta-commentary flag, missing expansion, cross-reference accuracy, equation punctuation.

## Layer 3: Notation audit
Each symbol defined once. Consistent with reference papers. Display equations punctuated.

## Layer 4: Structure (C-group — per-paragraph reading)
3 numbered sections + no Conclusion. Setup:proof ≤ 1:2. Each Lemma preceded by motivation.

## Layer 5: LaTeX — tectonic 0 errors.

## Per-File B/C-Group Template

### B-group (per sentence)
| # | Check |
|---|-------|
| B1 | Meta-commentary? ("this identity is central", "key innovation") |
| B2 | Grammar subject (Thus/Now/Let/we/If/By/The/This) |
| B3 | Redefinition of earlier symbol? |
| B4 | Missing bracket expansion? (claim without derivation) |
| B5 | Cross-reference accurate? |
| B6 | Motivation sentence before Lemma? |
| B7 | Display equation punctuation? |

### C-group (per paragraph)
| # | Check |
|---|-------|
| C1 | One thing per paragraph? |
| C2 | First sentence previews content? — delete it |
| C3 | Delete first sentence test — argument still works? |
| C4 | Moreover/Furthermore connector between paragraphs? — delete it |

## Verification Rules
- "whence" is always allowed (KK style)
- "i.e." allowed ONLY for symbol substitution, NOT for prose explanation
- Ou DOES use "This paper is organized as follows" — previous skill claim wrong
- "vanishes at (u,v)" meaning "projects to zero" is acceptable

## Post-Fix Re-Scan
After all fixes: re-run Layer 1 grep. Re-compile. Check for new undefined refs.
