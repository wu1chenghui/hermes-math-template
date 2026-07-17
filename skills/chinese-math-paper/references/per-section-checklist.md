# Per-Section Translation Checklist

After translating each section (Phase 1), verify ALL items before proceeding.

## Mandatory checks

- [ ] Math content matches English version exactly
- [ ] LaTeX structure unchanged
- [ ] All `\label{...}` preserved
- [ ] All formulas preserved (no formula-level translation)
- [ ] All `\ref`, `\eqref`, `~\\ref` preserved
- [ ] Theorem/Lemma/Equation numbering unchanged
- [ ] Lean theorem names NOT translated (keep `coeffOf_cond`, etc.)
- [ ] Glossary terms used consistently with glossary.tex
- [ ] Paragraph-by-paragraph correspondence maintained

## Style checks (Phase 1)

- [ ] No mechanical "得到" (use 定义为/引出/给出 instead)
- [ ] No mechanical "表示" (use 刻画/描述/对应于 instead)
- [ ] No mechanical "作为" (use 充当/成为 instead)
- [ ] No "这是因为" in body (use 其中/由于 instead)
- [ ] No "本文" in body sections (§2+)
- [ ] First occurrence of each term has parenthetical English
- [ ] Introduction uses Chinese paper conventions (本文……/本文的结构如下)

## Phase 2 checks (global, after all sections done)

- [ ] Cross-section terminology 100% consistent
- [ ] No residual English-style phrasing
- [ ] Sentence patterns unified across sections
- [ ] Long sentences adjusted for Chinese readability
- [ ] All forward references resolve correctly
