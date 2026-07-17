# Terminology Validation Methodology

> How to validate a Chinese math glossary when web search is unavailable or
> returns poor results for Chinese academic terms.

## When web search works poorly

Common on DDGS (DuckDuckGo): Chinese mathematical terminology searches
often return physics/chemistry results, unrelated content, or no results.
SearXNG also cannot extract from arxiv, raw.githubusercontent.com, etc.

Do NOT spend many rounds retrying the same search — if 3 queries return
no relevant math results, switch to logical validation.

## Three-tier classification

Categorize every glossary entry into one of three tiers:

### Tier A: Standard terms with authoritative Chinese mathematical precedent

These have known, fixed translations used in Chinese textbooks,
Wikipedia, and published papers. Examples:
- derivation → 导子 (standard in Lie algebra theory)
- commutator → 换位子 (zh.wikipedia.org/wiki/李代數 uses this)
- nilpotent → 幂零 (baike.baidu.com has "幂零李代数" entry)
- trivial → 平凡 ("平凡子群", "平凡表示" widely used)
- involution → 对合 (standard)
- automorphism → 自同构 (standard)

**Verification**: cite the source (Wikipedia, Baidu Baike, standard textbook).

### Tier B: Terms derivable from source literature

These are new terms whose English origin comes from a specific paper.
The Chinese follows the same derivation logic. Examples:
- δ-derivation → δ-导子 (Filippov introduced δ-derivations; 导子 = derivation)
- 1/2-derivation → 1/2-导子 (δ=1/2 case; preserve fraction notation)
- transposed Poisson structure → 转置Poisson结构 (proper noun + 结构)

**Verification**: confirm the English source uses the term, confirm the
Chinese follows the same naming pattern.

### Tier C: Paper-specific novel terms

These are coined by the paper being translated. No web verification
is possible — validate by logic:
1. Is the Chinese grammatically correct?
2. Does it collide with existing standard mathematical terminology?
3. Does it follow Chinese mathematical naming conventions?

Examples from our paper (pre-debranding; these were later removed as named terms
in English v3, 2026-07-11):
- chain bridge → 链桥 (was defined in §1; now replaced by \eqref reference)
- width reduction → 宽度化简 ("化简" preferred over "归约" for math context)
- image restriction → 图像限制 ("图像" = image, "限制" = restriction)
- centered decomposition → 中心化分解 (retained — a genuine mathematical definition)

## Pitfalls when a glossary file exists

An existing glossary file (e.g., from a previous version of the paper)
is a LIABILITY, not an asset:

- v1/v2 glossary may contain terms deleted in v3 (e.g., "Psi operator",
  "equation space", "T-family generators")
- Terminology conventions may have changed (e.g., "归约" → "化简")
- "half-Leibniz condition" was in the old glossary but deleted from v3

**Rule**: always derive the glossary fresh from the CURRENT English version.
Read every .tex file and extract all terms. Then cross-check against any
existing glossary — but never start from the old glossary.

## Terminology consistency audit

After translation is complete, run a global search for ALL Chinese terms
across all translated files. Verify:
1. Each English term maps to exactly one Chinese term (no synonyms)
2. Each Chinese term maps back to exactly one English concept
3. "得到" count is ≤ 2 (abstract only; body text uses 给出/可得)
4. "迫使" count is 0
