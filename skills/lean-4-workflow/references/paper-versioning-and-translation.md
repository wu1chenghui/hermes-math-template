# Paper Versioning & Chinese Translation Methodology

## Three-version strategy

When the user transitions from proof development to paper writing, maintain
three versions with identical mathematics but different audiences:

| Version | Folder | Audience | Key Features |
|---------|--------|----------|-------------|
| Journal (投稿版) | `paper-cn/` | Peer reviewers | Frozen, minimal, formal |
| Teaching (教学版) | `paper-cn-detailed/` | Graduate students | Three-layer navigation (section→subsection→lemma motivation) |
| Lecture (讲义版) | `paper-cn-lecture/` | Undergraduates | Prerequisites, warm-up examples, discovery/strategy/pitfall boxes |

The Lecture version should include:
- Prerequisite review (linear map dimension, rank-nullity, derivation basics)
- A full warm-up computation (e.g., dim Der(N_n) = 2n-1) to demonstrate methodology
- Concrete examples throughout (n=5 used pervasively)
- Discovery boxes (🔍 "Why did we think of this?")
- Strategy boxes (📋 "Current strategy")
- Pitfall boxes (⚠ "Common misunderstanding")
- Parameter tally tables
- Reading recommendations (three-pass method)

## Chinese translation workflow (10 principles, frozen)

Principles in glossary.tex v3:

0. English term in parentheses on first occurrence; Chinese only thereafter.
1. Lean theorem/file names preserved in English.
2. "force" → context-dependent (推出/必有/可知), never "迫使".
3. chain bridge → 链式桥接 (first) / 链桥 (thereafter).
4. centered → 中心化 (centered) (first) / 中心化 (thereafter).
5. cocycle → cocycle preserved, never "余循环".
6. Two-phase: Phase 1 (faithful) then Phase 2 (polish).
7. Rewrite in Chinese math paper style, not sentence-by-sentence.
8. Avoid mechanical "得到/表示/作为"; use 定义为/刻画/对应于/充当.

Phase 1: Keep all LaTeX, labels, formulas, references, theorem numbers.
Phase 2: Remove English flavor → eliminate 得到/迫使/表示/作为, unify connectors.
Phase 3 (Narrative Reconstruction): Add navigation sentences without changing math.

## Chinese math paper conventions

Style references: 《中国科学》《数学学报》《数学年刊》

Forbidden in translation:
- 得到 → use 给出/确立/定义/推出
- 迫使 → use 推出/使得/压缩
- 表示 → use 刻画/对应于 (when meaning "represent" not "express")
- 作为 → use 充当/以...形式
- 纸质证明 → 本文中的证明
- 装配 → 综合前述结果
- 编码 → 刻画

Preferred patterns:
- "In this paper, we..." → "本文……"
- "The main result is..." → "本文的主要结果如下。"
- "We now prove..." → "现在证明……" (not "接下来我们将详细证明……")
- Introduction: use 本文; body sections: use 定义/注意/下面证明/我们首先证明

## LaTeX compilation

Engine: tectonic (XeTeX-based). For Chinese: `\usepackage{ctex}` replaces `\usepackage[utf8]{inputenc}`.

Thesis formatting standard:
- A4, margins 2cm, binding offset 0.5cm
- 五号宋体 body, 小二号黑体 title
- 【摘 要】小五号黑体 + 小五号楷体
- 【关键词】小五号黑体 + 小五号楷体
- Section numbering: 1 → 1.1 → 1.1.1
- Formula numbering: (section--equation) e.g., (2-3)
- Use `\setlength{\emergencystretch}{2em}` for overfull tolerance
- `{\sloppy ... \par}` for stubborn paragraphs with wide inline math
