# Sentence Structure Metrics (Ou Imitation)

When auditing a proof section (equivalent to Ou's §3), check these 8 quantifiable
targets. Source: forensic analysis of Ou-Wang-Yao (2007) §3.

| Metric | Target | Check Method |
|--------|--------|-------------|
| Pure math sentence ratio | ≥ 63% | Delete sentences with no mathematical content |
| Pure navigation sentences | ≤ 2 total | Only allowed at section openings |
| Grammar subject variety | ≥ 5 types | let/suppose, thus/so, by/applying, this/these, now, recall/note, choose, step |
| Max subject share | ≤ 15% | No single subject type exceeds 1/7 of sentences |
| Consecutive same subject | 0 | No two consecutive sentences start with the same subject word |
| Short:Medium:Long ratio | ≈ 4:3:3 | Short <15 words, Medium 15-25, Long >25 |
| Long→Short breathing | Required | Every >25 word sentence followed by at least one <15 word sentence |
| Deep→Flat breathing | Required | Every clause depth >5 sentence followed by at least one depth ≤1 sentence |

## Target distribution for a 60-sentence §3

- Pure math: 38 sentences + Mixed: 20 sentences + Pure navigation: 2 sentences
- Short: 25, Medium: 20, Long: 15
- 8+ subject types, each ≤ 9 sentences

## Clause depth

Depth = commas + that-clauses + which-clauses + if-clauses + since-clauses + whence-clauses.
- Short sentences (<15w): avg depth 1.1 (almost flat)
- Long sentences (>25w): avg depth 8.6 (deeply nested)
- Pattern: deep → flat → deep → flat (cognitive breathing)
