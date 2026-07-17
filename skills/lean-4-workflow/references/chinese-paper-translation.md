# Chinese Paper Translation Workflow

Established 2026-07-04 during the full Chinese translation of the
HalfDer paper (8 sections + appendix).

## Three-version system

| Version | Path | Purpose |
|---------|------|---------|
| Journal | `paper-cn/` | Submission-ready, frozen |
| Teaching | `paper-cn-detailed/` | 3-layer navigation (section overview + subsection motivation + lemma motivation) |
| Lecture | `paper-cn-lecture/` | Undergraduate self-study (discovery boxes, examples, pitfalls) |

All three share identical mathematics; differ only in narrative depth.

## Translation principles (frozen, 10 rules)

0. English term in parentheses on first occurrence; Chinese only thereafter.
1. Lean theorem/file names preserved in English.
2. "force" → context-dependent (推出/必有/可知), never "迫使".
3. chain bridge → 链式桥接 (first) / 链桥 (thereafter).
4. centered → 中心化(centered) (first) / 中心化 (thereafter).
5. cocycle → cocycle preserved, never "余循环".
6. Two-phase: Phase 1 (faithful) then Phase 2 (polish).
7. Rewrite in Chinese math paper style, not sentence-by-sentence translation.
8. Avoid mechanical "得到/表示/作为"; use 定义为/刻画/对应于/充当.

## Key terminology

| English | Chinese |
|---------|---------|
| half-derivation | 1/2-导子 |
| constraint reduction | 约束归约 |
| chain bridge | 链式桥接 / 链桥 |
| width reduction | 宽度归约 |
| diagonal propagation | 对角传播 |
| image restriction | 像限制 |
| endpoint reduction/coupling | 端点归约/端点耦合 |
| parameter count | 参数统计 |
| evaluation matrix | 求值矩阵 |
| centered derivation | 中心化导子 |
| boundary cocycle | 边界 cocycle |
| a/b/c-channel | a/b/c-信道 |
| self-pair | 自配对 |
| half-Leibniz morphism | 半-Leibniz 映射 |
| currying | 柯里化 |

## Lecture notes pedagogical modules

For undergraduate audience, include:
- 🔍 Discovery boxes ("why did we think of this?")
- ⚠ Pitfall boxes (common misunderstandings)
- 📋 Strategy boxes (current plan)
- 💡 Motivation markers (思路)
- Concrete n=5 examples throughout
- Test-point tables for linear independence proofs
- "Breaking sticks" analogy for width reduction
- Bracket quick-reference with domino analogy

## Glossary management

File: `paper-cn/glossary.tex` (v3, frozen)
Covers ~250 terms across 10 categories.
Modification requires user confirmation (A/B/C classification).
