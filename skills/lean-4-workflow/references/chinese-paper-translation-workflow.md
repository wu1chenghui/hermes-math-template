# Chinese Paper Translation Workflow

Established during the `e/` project paper translation (2026-07-03).

## Three-version architecture

```
paper-cn/           Journal Version   投稿版  已冻结
paper-cn-detailed/  Teaching Version  教学版  三层导航
paper-cn-lecture/   Lecture Notes     讲义版  预备知识+热身+发现层
```

## 10 Frozen Translation Principles

0. English term in parentheses on first occurrence; Chinese only thereafter.
1. Lean theorem/file names preserved in English.
2. "force" → context-dependent (推出/必有/可知), never "迫使".
3. chain bridge → 链式桥接 (first) / 链桥 (thereafter).
4. centered → 中心化 (centered) (first) / 中心化 (thereafter).
5. cocycle → cocycle preserved, never "余循环".
6. Two-phase: Phase 1 (faithful) then Phase 2 (polish).
7. Rewrite in Chinese math paper style, not sentence-by-sentence translation.
8. Avoid mechanical "得到/表示/作为"; use 定义为/刻画/对应于/充当.

## Key Terminology (final, frozen)

| English | Chinese |
|---------|---------|
| half-derivation | 1/2-导子 |
| constraint reduction | 约束归约 |
| chain bridge | 链式桥接 / 链桥 |
| width reduction | 宽度归约 |
| diagonal propagation | 对角传播 |
| image restriction | 像限制 |
| endpoint reduction / coupling | 端点归约 / 端点耦合 |
| parameter count | 参数统计 |
| evaluation matrix | 求值矩阵 |
| centered derivation | 中心化导子 |
| boundary cocycle | 边界 cocycle |
| a/b/c-channel | a/b/c-信道 |
| channel decomposition | 信道分解 |
| self-pair | 自配对 |
| half-Leibniz morphism | 半-Leibniz 映射 |
| currying | 柯里化 |
| rank-nullity theorem | 秩–零度定理 |

## Compilation

Engine: tectonic (XeTeX-based). Use `\usepackage{ctex}` for CJK support.
Paper size: A4, margins 2cm for thesis format.

## Per-section freeze discipline

Each section is frozen after user review. Do NOT revisit frozen sections
unless a mathematical error is discovered. Introduction (Section 1) is
particularly sensitive to over-polishing — limit to 3 revision rounds max.

## Common pitfalls

- `dim Der(N_n) ≠ 2n-1`. Correct: n(n-1)/2 + 2n - 3.
- Self-pairing does NOT prove image restriction. The argument is vacuous.
- "表示/得到/作为" should be avoided; use Chinese math paper alternatives.
- Channel → 信道 (not 通道). Evaluation → 求值 (not 评价).
