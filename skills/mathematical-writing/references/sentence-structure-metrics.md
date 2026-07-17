# Sentence Structure Metrics — Ou-Style Imitation Targets

> Extracted from forensic analysis of Ou-Wang-Yao (2007) §3.
> Use these metrics to audit a proof section after writing.

## Core Metrics

| # | Metric | Target | How to Check |
|---|--------|--------|-------------|
| 1 | Pure math sentences | ≥ 63% | Delete sentences with zero mathematical content |
| 2 | Pure navigation sentences | ≤ 2 total | Only at section start; never standalone |
| 3 | Grammatical subject types | ≥ 5 kinds | let/suppose, thus/so, by/applying, this/these, now, recall/note, choose, step |
| 4 | Max subject type share | ≤ 15% | No pattern exceeds 1/7 of sentences |
| 5 | Consecutive same subject | 0 | Never two sentences in a row with same subject type |
| 6 | Short:medium:long ratio | ≈ 4:3:3 | Short <15w, medium 15-25w, long >25w |
| 7 | Breath after long sentence | Required | Every >25w sentence followed by ≥1 sentence <15w |
| 8 | Breath after deep nesting | Required | Every depth>5 sentence followed by ≥1 sentence depth 0-1 |

## Target Distribution (60-sentence proof section)

- 38 pure math + 20 mixed + 2 navigation
- 25 short + 20 medium + 15 long
- 8+ subject types, each ≤ 9 sentences

## Clause Depth

- Overall average: 3.7 clauses per sentence
- Short sentences (<15w): avg depth 1.1 (nearly flat)
- Long sentences (>25w): avg depth 8.6 (deeply nested)
- Pattern: deep → flat → deep → flat (cognitive breathing)

## Subject Type Reference

| Type | Avg Length | Function |
|------|-----------|----------|
| recall/note | 11.8w | Quick side observation |
| now | 12.0w | Simple transition |
| this/these | 13.4w | Anaphoric reference |
| step header | 17.2w | Structural marker |
| by/applying | 18.3w | Method description |
| choose/define | 21.5w | Construction |
| let/suppose | 24.2w | Parameterization (workhorse) |
| thus/so | 24.7w | Conclusion (variable length) |
