---
name: chinese-math-paper
description: >-
  Write, translate, format, and expand Chinese mathematical papers.
  Covers glossary-driven terminology management, two-phase translation
  (faithful conversion → Chinese polish), thesis formatting (ctex),
  narrative reconstruction (Phase 3), and pedagogical lecture-note
  expansion with discovery/motivation/pitfall layers.
---

# Chinese Mathematical Paper Writing & Translation

> **When to load:** translating an English math paper to Chinese,
> formatting a Chinese math paper for submission/thesis,
> improving the narrative flow of a proof-heavy paper, or
> expanding a research paper into self-contained lecture notes.

## Communication language

Discussion and strategy in Chinese. Lean code, lemma names, and
mathematical notation stay in English. This is the user's strong
preference and is recorded in their profile.

## Translation Pre-Flight: English Compression Pass

> **Ou/KK style alignment** — after compression, run the full style-alignment
> checklist at `references/ou-kk-style-alignment.md`. It covers 8 specific
> patterns: parenthetical removal, roadmap deletion, case-label flattening,
> global-declaration compression, coined-term replacement, pattern-language
> removal, sub-case merging, and transition cleanup. These are the patterns
> that make a paper read like Ou-Wang-Yao (2007) or KK (2023) rather than
> a modern tutorial.

**Always compress the English source before translating.**  Chinese text is
~20% longer on the page than English; an uncompressed 6-page paper will blow
up to 7-8 pages after translation.  Ou-Wang-Yao (2007) and KK (2023) are the
style references.

### Compression targets (in priority order)

1. **Inline single-line equations.**  If a display equation (`\[...\]`) contains
   a single line, convert it to inline (`$...$`).  Only multi-line aligned
   equations and numbered equations should remain as display.

2. **Delete explanatory prose.**  Strike sentences that restate what an equation
   already shows (e.g., "This identity expresses the coefficients of X through
   the coefficients of Y").  Strike parenthetical justifications like "(this
   requires n≥5)" when the condition is already a global hypothesis.
   For a systematic audit of ALL explanatory parentheticals against reference
   papers (Ou/KK), see `references/parenthetical-audit.md`.  Target: 0.

   **Explanatory parentheticals** are a distinct sub-category: parenthetical
   phrases that embed proof reasoning (e.g., "(only the bracket-right term
   contributes, via …)", "(using d₃=d₁ from above)").  Ou (2007) and KK (2023)
   have **zero** such parentheticals.  Audit methodology and reference-paper
   comparison data in `references/explanatory-parenthetical-audit.md`.
   Key rule: **never absorb into the sentence** — a `via` clause tacked onto
   a sentence is still an explanatory parenthetical.  Just delete it.  The
   reader can verify the computation themselves.

3. **Drop construction verifications.**  Use "one checks" / "it is easy to
   verify" instead of showing every bracket expansion.  Ou uses this pattern
   for ALL his standard derivations.

4. **Merge signpost paragraphs.**  Delete "We distinguish three cases", "First,
   we prove...", "Five additional derivations are supported at the Dynkin
   endpoints."  The case labels and lemma statements speak for themselves.

5. **Compress bracket calculations.**  "The bracket-left term contributes -c_k"
   instead of showing all three terms with two that are zero.

6. **Delete unused properties.**  "This ideal is abelian and satisfies" → "This
   ideal satisfies" if the abelian property is never used in proofs.

### Density comparison

| Metric | Ou (2007) | Our v3 (after compression) |
|--------|-----------|---------------------------|
| Pages | 6 | 5 |
| Font | 9pt single-col | 10pt 2-col |
| Display eqs in main proof | 0 | ~25 |
| Construction verification | "it is easy to check" | 1-2 line checks |

Before translating, ensure the English original follows mathematical
paper conventions (not lecture-note style). Full audit checklist at
`references/english-paper-conventions.md`. Key rules:
- No named proof techniques (cite lemmas/equations by number)
- No source/target/width as informal terminology
- No numbered Step 1/2/3/4 frameworks
- No ornamental observations not used in proofs
- Global condition declaration after Theorem 1.1
- KK (arXiv:2305.00727) is the style reference for 1/2-derivation papers

---

## Core Principle: English is the Master

The English version is the canonical reference. The Chinese version
must be a **faithful mapping**, not a rewrite:

- Every Chinese sentence should correspond uniquely to an English sentence.
- Do NOT reorder proofs, add explanations, or rewrite logic for fluency.
- Formulas, labels, theorem/lemma/corollary/equation numbers stay unchanged.
- Lean lemma names stay in English (never "translate" `coeffOf_cond`).

### Editing workflow: plan first, then act

When the user asks to modify the paper (delete something, restructure,
change terminology), do NOT immediately execute.  First:
1. Read the full context around each target.
2. Present a plan showing what changes and why.
3. Re-examine: "Is this plan optimal, or is there a better approach?"
4. Only after user confirmation, execute.
The user explicitly prefers this sequence for all paper-editing decisions.

---

## Glossary-Driven Translation

**Always establish a glossary BEFORE translating.** Read the full paper,
extract all terminology (~100-250 terms), and freeze the glossary.

### Glossary format

**CRITICAL: Always derive the glossary fresh from the CURRENT English version.**
An existing glossary file from a previous version is a liability — it
contains stale terms that may have been deleted, and terminology conventions
may have changed (e.g., "归约" → "化简"). Read every .tex file, extract all
terms, then cross-check against any existing glossary.

For validating translations when web search returns no Chinese math results
(common with DDGS/SearXNG), see `references/term-validation.md`.

```latex
% English term                    Chinese translation
% half-derivation                 1/2-导子
% constraint reduction            约束归约
% image restriction               像限制
```

### Key translation conventions (frozen for this user)

| English | Chinese | Notes |
|---------|---------|-------|
| half-derivation | 1/2-导子 | KK uses "1/2-derivation"; "半导子" has semigroup connotation — avoid |
| strictly upper triangular | 严格上三角 | Standard |
| Lie algebra | 李代数 | Standard |
| bracket | 李括号 | Standard |
| matrix unit | 矩阵单位 | Standard |
| Kronecker delta | Kronecker delta | Retain English |
| coordinate projection | 坐标投影 | Standard |
| central | 中心/中心元 | Standard |
| ideal | 理想 | Standard |
| abelian | 交换的 | Alternative: 阿贝尔的 |
| centered / centered decomposition | 中心化 / 中心化分解 | Defined in paper; add "(centered)" on first use |
| trivial derivation | 平凡导子 | Standard; used in Chinese math papers |
| simple-root derivation | 单根导子 | Standard Lie theory term |
| boundary derivation | 边界导子 | Descriptive |
| commutator | 换位子 | Standard |
| Dynkin involution | Dynkin对合 | Dynkin retained as proper noun |
| parameter tally | 参数统计 | Standard |
| commuting pair | 交换对 | Standard |
| linear map | 线性映射 | Standard |
| linearly independent | 线性无关 | Standard |
| automorphism | 自同构 | Standard |
| commutative ring | 交换环 | Standard |
| δ-derivation | δ-导子 | Standard |
| transposed Poisson structure | 转置Poisson结构 | Standard |
| upper bound / lower bound | 上界 / 下界 | Standard |
| dimension / dim | 维数 | Standard |
| force (verb) | 推出 / 必有 / 可知 | NEVER 迫使 |
| obtain/yield/get | 给出 / 引出 | NOT 得到 (mechanical overuse) |
| represent | 刻画 / 对应于 | NOT 表示 |
| serve as | 充当 / 给出 | NOT 作为 |

Note (2026-07-11): After English v3 debranding, the following terms are
no longer used as named concepts: chain bridge, width reduction,
diagonal propagation, image restriction, endpoint reduction, channel
(a/c-channel), cross-endpoint constraint. The terms "evaluation matrix",
"boundary cocycle", and "syzygy" do not appear in v3.

### coeff / coeffOf / coefficient — three-level distinction

| Lean | Chinese | Scope |
|------|---------|-------|
| `coeff` | 系数函数 | Lean internal: the raw 4-index function `ℕ⁴ → F` |
| `coeffOf` | 系数投影 | Lean API: totalized wrapper, returns 0 for invalid indices |
| coefficient | 系数 | Math: `π_{uv}(T(E_{ij}))`, the projection value |

NEVER conflate these. In prose, use 系数投影 for `coeffOf` on first occurrence
with parenthetical `(coeffOf)`. In the Lean-formalization footnote ($\S$2),
explain that `coeffOf` is a totalized wrapper.

### Lean-style phrasing — FORBIDDEN in body text

| Forbidden | Use instead |
|-----------|------------|
| 返回 (return) | 定义... / 给出... |
| 调用 (call) | 应用... / 作用于... |
| 包装器 (wrapper) | footnote only; body uses 系数投影 directly |
| 接受参数 | 以...为输入 / 作用于 |
| 迫使 | 推出 / 给出 / 要求 / 使 |

These only appear in the Lean-formalization footnote ($\S$2), never in body text.

### Punctuation

Chinese version uses **English punctuation** (. , not 。，). This is the user's
stated preference. Apply `.replace('。', '.')` and `.replace('，', ',')` as a
final pass after all translation is complete.

**GitHub README rendering**: when math expressions appear near CJK full-width punctuation
in GitHub markdown, add ASCII spaces around `$` delimiters. Full recipe in
`halfder-nn-repo-setup` skill, `references/github-rendering-pitfalls.md`.

### GitHub README rendering with CJK
When publishing Chinese math on GitHub, full-width CJK punctuation touching
`$` delimiters prevents MathJax from rendering. Always add ASCII spaces:
`：$...$，` → `： $...$ ，`. Full guide: load `paper-engineering` skill and
see `references/github-markdown-math.md`.

### Lean names

All Lean theorem names, file names, and `\texttt{}` identifiers stay in English:
`coeffOf_cond`, `finrank_halfDer`, `HalfDerivation`, `Phi`, `MatIdx`, etc.
NEVER translate these.

### Pre-translation audit: remove self-coined technique names

Before translating, audit the English paper for unnecessary coined
terminology. Compare against key references in the field (e.g., KK 2023,
Filippov 1998) and remove any names they would not use. Replace named
techniques with lemma/equation number references. Full methodology and
findings in `references/debranding-audit.md`.

### Translation freeze workflow

**WARNING:** After compressing the English paper, the translation plan
MUST be completely rewritten. See `references/compression-workflow.md` for
the 5-round iterative compression methodology and common redundancy patterns.

When translating an English paper that has a Lean-formalized proof:
1. **Freeze English first.** Confirm all patches applied, final compilation
   passes with 0 errors. Tag as "translation baseline".
2. **Create a fresh `paper-cn-v2/` folder.** Copy `main.tex` (ctex preamble)
   and `glossary.tex` from existing Chinese version. Create empty stubs for
   each section.
3. **Translate sentence-by-sentence from frozen English.** Do NOT reuse old
   Chinese translations section-by-section. Even sections that appear
   unchanged may have accumulated terminology drift.
4. **Translation order: highest-risk first.** §5 (most changes) → §7
   (new lemmas) → §2 (new footnotes) → §1 → §3 → §4 → §6 → §8.
5. **Add new glossary terms BEFORE translating** the section that introduces
   them.

### Translation principles (10 rules)

0. English term in parentheses on first occurrence; Chinese only thereafter.
1. Lean theorem/file names preserved in English.
2. "force" → context-dependent, never "迫使".
3. centered → 中心化（centered）(first) / 中心化 (thereafter).
4. cocycle → preserved, never "余循环".
5. Two-phase: Phase 1 (faithful) then Phase 2 (polish).
6. Rewrite in Chinese math paper style, not sentence-by-sentence.
7. Avoid mechanical "得到/表示/作为".

---

## Two-Phase Translation Workflow

### Phase 1: Faithful Conversion

- Keep all math, labels, formulas, references, LaTeX structure unchanged.
- No additions or deletions of any explanation.
- Section-by-section, freeze each before moving to the next.

### Phase 1.5: Per-Section Compile + Audit

After EACH section, before moving to the next:
1. Compile the PDF to catch LaTeX issues immediately.
2. Run the Chinese Translation Audit Checklist
   (`references/per-section-checklist.md`):
   - Mechanical checks (0 `迫使`, flag excessive `得到`)
   - Math consistency (formulas, labels, references unchanged)
   - Terminology consistency (glossary compliance)
   - Chinese math style (no translation-ese)
   - Proof logic (causality preserved, no ambiguity)
3. Fix ALL issues before moving to the next section.
Do NOT batch-translate multiple sections without auditing in between.

### Phase 2: Unified Chinese Polish

- Done AFTER all sections are translated.
- Remove English flavor, unify sentence patterns and connectors.
- Fix overfull hboxes.
- Check terminology consistency across all sections globally.

---

## Phase 3: Narrative Reconstruction

When the paper feels "fragmented" to read, add a **narrative layer**
without changing any mathematics:

1. **Section Opening**: why this section exists, what it achieves.
2. **Subsection Transition**: why we're proving THIS now, after the last step.
3. **Lemma Motivation**: what problem this lemma solves, why it's needed.

The goal: a first-time reader always knows WHY the author is doing each step.

### Compression staircase (universal pattern)

For constraint-reduction proofs, display the reduction order:

```
所有线性映射 (d² 维)
    ↓ \eqref{eq:halfder} 约束
源压缩：只留下相邻源
    ↓
对角压缩：对角线归零
    ↓
目标压缩：像限制到 3 维理想
    ↓
端点压缩：消去内部冗余
    ↓
跨端点约束：F_eq 耦合两端
    ↓
参数统计：n+5
```

---

## Thesis Formatting (Chinese `ctex`)

For Chinese thesis/dissertation formatting:

```latex
\documentclass[a4paper]{ctexart}
\usepackage[top=2cm, bottom=2cm, left=2cm, right=2cm,
            bindingoffset=0.5cm]{geometry}
\linespread{1.0}

\ctexset{
  section = {format = \heiti\zihao{4}\bfseries},
  subsection = {format = \heiti\zihao{-4}\bfseries},
  subsubsection = {format = \heiti\zihao{5}},
}

% Formula numbering by chapter: (1-1), (2-3)
\numberwithin{equation}{section}
\renewcommand{\theequation}{\thesection--\arabic{equation}}

% Theorem environments
\newtheorem{theorem}{定理}[section]
\newtheorem{lemma}[theorem]{引理}
\newtheorem{corollary}[theorem]{推论}
\newtheorem{definition}[theorem]{定义}
\newtheorem{remark}[theorem]{注记}
```

Compile with `tectonic` (XeTeX-based, auto-downloads packages).

### Word template → ctexart conversion

When the user provides a `.docx` format reference, extract all formatting
parameters before writing LaTeX. Full workflow in
`references/docx-format-extraction.md`. Key pitfalls:
- Add `\pagestyle{empty}` after `\begin{document}` — ctexart defaults to
  section-name running headers, but thesis format has none.
- Use `\ctexset{section/aftername=\hspace{0pt}}` to remove space between
  section number and title (for "1标题" format).
- Use `\newtheoremstyle` from amsthm when theorem head fonts differ from
  amsthm defaults (common in Chinese thesis templates).
- Word Normal style font size ≠ body text font size — always verify with
  XML or check the Normal style default rather than trusting run-level `font.size`.
- **楷体加粗不生效** — `\kaishu\bfseries` produces no visible bold in ctex.
  Fix: `\setCJKfamilyfont{kaishu}{KaiTi}[AutoFakeBold={2.5}]`.  Full pitfall
  list at `references/ctex-pitfalls.md`.
- **LaTeX → Word conversion** — use pandoc with flattened input files and
  stripped ctex commands.  Math is preserved as OMML.  Full workflow at
  `references/pandoc-conversion.md`.

---

## Lecture Notes Expansion (Pedagogical Version)

When expanding a paper into self-contained lecture notes for undergraduates:

### Required layers (beyond the paper)

1. **Prerequisites section**: dim(Hom), rank-nullity, bracket computation
   quick-reference, source/width definitions, Dynkin diagram basics.
2. **Discovery boxes**: "Why did the author think of this step?"
3. **Strategy boxes**: "What's the current plan?"
4. **Pitfall boxes**: "Common misunderstanding: ..."
5. **Worked examples**: Pick a concrete small n (e.g., n=5) and trace
   every step through it.
6. **Visual definitions**: Source space, diagonal, target space, channel,
   cross-endpoint — define all five compression dimensions explicitly.
7. **Reading recommendations**: Three-pass reading guide.

### Pitfall: verify ALL factual claims

The lecture notes may contain "background knowledge" that the paper
doesn't depend on. Verify these independently. Example: a claim that
dim Der(N_n) = 2n-1 is WRONG for strictly upper triangular N_n;
the correct formula is n(n-1)/2 + 2n - 3.

### Tools for lecture notes

Use these environments:

```latex
\newenvironment{pitfall}
  {\medskip\hrule\medskip\noindent\textbf{⚠ 常见误解：}\par\medskip}
  {\medskip\hrule\medskip}

\newenvironment{discovery}
  {\medskip\hrule\medskip\noindent\textbf{🔍 为什么想到这一步？}\par\medskip}
  {\medskip\hrule\medskip}

\newenvironment{strategy}
  {\medskip\hrule\medskip\noindent\textbf{📋 当前策略：}\par\medskip}
  {\medskip\hrule\medskip}

\newcommand{\motivation}[1]{\medskip\noindent\textbf{💡 思路：} #1\medskip}
```

---

## Three-Tier Output Architecture

For this user's projects, three versions are typically produced:

```
paper-cn/           Journal Version   投稿版  Frozen, submission-ready
paper-cn-detailed/  Teaching Version  教学版  Three-layer navigation
paper-cn-lecture/   Lecture Notes     讲义版  Self-contained for undergrads
```

All share identical mathematics; differ only in narrative depth.

---

---

## XIII. Formatting Requirements — Chinese Math Journals

> Research conducted 2026-07-11. Methodology documented in
> `references/research-methodology.md`.
> Sources: 数学年刊A辑 征稿简则, GB/T 7713.2-2022,
> 中国科学:数学 LaTeX template, Zhihu 数学符号规范, general conventions.

### Punctuation: Use English "." (Critical)

Chinese math papers should use **English (half-width) punctuation** to avoid
the Chinese period "。" being confused with subscript "₀" (e.g., `E₀`, `φ₀`,
`x₀`). This is a widely recognized convention in Chinese mathematical writing.

| Punctuation | Use | Avoid |
|-------------|-----|-------|
| Period | `.` | `。` (confusable with subscript 0) |
| Comma | `,` | `，` |
| Colon | `:` | `：` |
| Semicolon | `;` | `；` |
| Parentheses | `()` | `（）` |

Note: Some Chinese math journals DO use Chinese punctuation. When targeting a
specific journal, check their requirements. For self-translation without a
target journal, English punctuation is the safer default.

### Spacing: Chinese ↔ English

Insert a space between Chinese characters and English/math inline elements:

| Correct | Wrong |
|---------|-------|
| `设 $F$ 为特征不为 $2,3$ 的域` | `设$F$为特征不为$2,3$的域` |

### Journal-Specific Requirements

#### 数学学报 (Acta Mathematica Sinica, Chinese Series)

| Requirement | Detail |
|-------------|--------|
| Abstract | 50–200 characters, self-contained (independently readable) |
| Keywords | 3–5 keywords |
| Classification | MR (2020) Subject Classification |
| Running head | ≤25 Chinese characters (书眉) |
| References | **All in English**, alphabetically by author surname |
| Section headings | 黑体 (bold), with size hierarchy for main/sub |
| Source format | Not specified publicly; likely CCT or LaTeX |

Key difference from 数学年刊: references must be in English (even for Chinese
papers) and alphabetically ordered, not sequentially numbered.

#### 数学年刊 A辑 (Chinese Annals of Mathematics, Series A)

| Requirement | Detail |
|-------------|--------|
| Paper size | A4 |
| Line spacing | Double (两倍行距) |
| Source format | **CCT** (not standard LaTeX; ctexart for drafting) |
| Chinese abstract | ≤200 characters |
| Keywords | 3–8 keywords |
| Classification | MR (2020) + 中图分类号 |
| English metadata | English title, abstract, keywords required alongside Chinese |
| Author names | 姓在前，名在后 (surname first) |
| References | Sequential numbering, specific Chinese format |
| Template | Available for download from 数学年刊 website (login required) |

#### 中国科学:数学 (Scientia Sinica Mathematica)

| Requirement | Detail |
|-------------|--------|
| Document class | `SCIA2023cn` (custom, not ctexart) |
| Chinese abstract | 100–200 characters |
| English abstract | 100–200 words |
| Keywords | 3–8, Chinese: `\quad` separated; English: comma separated |
| MSC | Both Chinese `\MSC{}` and English `\enMSC{}` required |
| Reference style | **Author surname alphabetical**, `\cite{}` inline, `\upcite{}` superscript |
| DOI | Recommended for each reference |
| Formula | Use `align` environment (preferred); avoid `equation`/`eqnarray` |
| Theorem style | Custom blue bold heiti (黑体), 5号 font |
| Theorem environments | 定理/定义/引理/推论/命题/猜想/注/例/问题/假设/条件/性质 |
| Figure caption | `\cnenfigcaption{中文}{英文}` |
| Table caption | `\cnentablecaption{中文}{英文}` |
| Title format | `\title{短标题}{完整标题}` + `\entitle{English short}{English full}` |
| Author format | `\author[n]{姓名}{邮箱}{标记}` + `\enauthor[]{English name}{}` |
| Address format | `\address[n.]{地址, 城市 邮编}` |
| Funding | `\Foundation{基金名称(批准号: XXX)}` |
| Acknowledgments | `\Acknowledgements{...}` |
| Appendix | `\begin{appendix}...\end{appendix}` |
| English title page | `\makeentitle` auto-generates |
| Template source | GitHub: goblinunde/LaTex-Resource |

#### CCT vs ctex

`CCT` (Chinese TeX) is a specialized format used by some older Chinese math
journals. It is NOT the same as `ctex`. For drafting, use `ctexart`. For
final submission to CCT-requiring journals, conversion would be needed.

### GB/T 7713.2-2022 — Key Provisions for Math Papers

*Extracted from the full standard text (2026-07-11).*

#### Formula Formatting (§5.7)

| Provision | Detail |
|-----------|--------|
| Formula display | Important formulas (with equation numbers) should be centered on their own line |
| Formula punctuation | **Allowed**: "居中排数学式的结尾，允许按其在行文中的语法关系添加标点符号" |
| Line breaking | Break at =, ≈, ≠, >, <, ≥, ≤ or +, −, ×, / operators; don't repeat operator on next line |
| Variable font | Variables in *italic*, constants and operators in upright (per GB/T 3102.11) |
| Inline vs display | Short formulas inline; numbered/large formulas displayed |

#### Typography (Appendix B)

| Element | Font Size | Font Family |
|---------|:---:|------|
| Chinese title | 小2号 (≈18pt) | 黑体 (Hei/bold) |
| Author names | 小4号 (≈12pt) | 楷体 (Kai) |
| Abstract, keywords | 小5号 (≈9pt) | 仿宋 (Fangsong) for content |
| **Section heading** | **小4号 (≈12pt)** | **黑体** |
| Subsection heading | 5号 (≈10.5pt) | 黑体 |
| **Body text** | **5号 (≈10.5pt)** | **宋体 (Song)** |
| Figure/table captions | 小5号 (≈9pt) | 黑体 |
| References | 小5号 (≈9pt) | 宋体 |

#### Abstract (§4.2.3)

| Type | Recommended length |
|------|:---:|
| 报道性 (informative) | ~400 characters |
| 报道/指示性 (mixed) | ~300 characters |
| 指示性 (indicative) | ~150 characters |

#### Other Provisions

| Provision | Detail |
|-----------|--------|
| Title length | ≤25 characters (§4.2.1) |
| Keywords | 3–8 per paper (§4.2.4) |
| Section numbering | Arabic numerals, ≤4 levels, dot-separated (§5.2.2) |
| Reference style | Sequential or author-year, consistent throughout (§4.3.6) |
| Paper size | A4 recommended (§5.1) |
| Page numbering | Continuous Arabic numerals (§5.8) |

#### Implications for ctexart Setup

The GB standard's body text size (5号 ≈ 10.5pt) differs from typical
Western math paper conventions (10–12pt). For ctexart:

```tex
\documentclass[5hao,a4paper]{ctexart}  % 5号 = ~10.5pt body text
% Or explicitly:
\documentclass[10.5pt,a4paper]{ctexart}
\setmainfont{Times New Roman}  % Western text
% ctex handles Chinese font (宋体 for body, 黑体 for headings) automatically
```

### Section & Theorem Naming (Standard)

| English | Chinese |
|---------|---------|
| Theorem | 定理 |
| Lemma | 引理 |
| Corollary | 推论 |
| Proposition | 命题 |
| Definition | 定义 |
| Remark | 注 |
| Proof | 证明 |
| Abstract | 摘要 |
| Introduction | 引言 |
| Preliminaries | 预备知识 |
| References | 参考文献 |

### Remaining Research Items

See also `references/writing-style-comparison.md` for quantitative and qualitative
comparison of writing style across reference papers (Ou, KK, Ghimire-Huang, Yusupov)
and our paper v3.

| # | Topic | Priority | Status |
|---|-------|:---:|:---:|
| 1 | 应用数学学报 requirements | Low | Not yet searched |
| 2 | CCT format specifications | Low | 数学年刊 only |

---

## XIV. Language Style — Chinese Math Prose

> **English paper writing conventions**: see `references/paper-writing-conventions.md`
> for a 10-point checklist distilled from comparing our paper against
> Ou (2007), KK (2023), Ghimire-Huang (2016), and Yusupov et al. (2025).
> These apply to the English source before translation.

### English Paper Style Reference

Before translating, consult `references/english-paper-style-audit.md` for a
quantitative comparison of writing styles across Ou-Wang-Yao (2007), KK (2023),
Ghimire-Huang (2016), and Yusupov et al. (2025). Key findings:
- "thus" density in our paper is 5× higher than all reference papers.
- Ou uses informal language ("get", "see", "easy to check") extensively.
- All reference papers avoid coined technique names, "source"/"target", and
  proof-method descriptions in abstracts.
- Named classification categories for derivations are standard (Ou does it too).

### Standard Sentence Patterns

| Pattern | Usage | Example |
|---------|-------|---------|
| 设...则... | Let...Then... | 设 $F$ 为域, 则... |
| 由...可得 | From...we obtain | 由引理3.1可得... |
| 故 | Thus | 故 $a=0$. |
| 即 | i.e. | 即 $\varphi$ 是中心化的. |
| 记...为... | Denote...by... | 记 $N(n,F)$ 为严格上三角矩阵. |
| 下证... | We now prove... | 下证 $\dim\Delta=n+5$. |
| 证毕 | QED | Auto-generated by \proof |
| 同理 | Similarly | 同理可证... |
| 不妨设 | WLOG | 不妨设 $n\ge5$. |
| 反之 | Conversely | 反之, 若... |
| 对任意... | For all... | 对任意 $i<j$... |
| 存在... | There exists... | 存在 $s\in F$... |
| 若...则... | If...then... | 若 $u=1$, 则... |

### Words to Avoid or Use Sparingly

| Word | Note |
|------|------|
| 显然 | "Obviously" — only for genuinely trivial facts |
| 易证 | Avoid in formal papers |
| 值得注意的是 | Delete — all facts have equal weight |

### Style Principles

1. **Short sentences**: Single-idea sentences preferred
2. **Formula density**: Prefer formulas over prose
3. **Impersonal tone**: Limit "我们"; prefer declarative
4. **No narrative**: State facts, don't describe discovery
5. **Consistent connectors**: Use same transitions throughout

### English → Chinese Translation Map

| English Pattern | Chinese Pattern |
|-----------------|-----------------|
| "We prove that..." | "下证..." |
| "It follows that..." | "由此可得..." |
| "By Lemma X, ..." | "由引理X, ..." |
| "Applying ... gives" | "将...作用于...得" |
| "Since ... we have" | "因...故..." |
| "Thus / So" | "故" |

## Reference Files

- `references/reference-verification.md` — Reference verification checklist: arXiv → published version, page numbers, issue numbers, orphan bibitems.
- `references/ctex-pitfalls.md` — ctex排版陷阱: 楷体加粗AutoFakeBold, 节编号空格, 页眉显示, 字体共存.
- `references/pandoc-conversion.md` — Pandoc LaTeX→Word 转换: 合并\input文件, 清理ctex命令, OMML公式保留.
- `references/docx-format-extraction.md` — Extracting formatting parameters from a .docx
  reference file using python-docx, and translating them to ctexart LaTeX preamble.
  Includes ctexset configuration, theorem style customization, and common pitfalls.
- `references/english-paper-style.md` — English mathematical paper writing conventions:
  what a research paper should and should not contain, derived from comparison
  with Kaygorodov-Khrypchenko, Ghimire-Huang, Yusupov et al., and Ou-Wang-Yao.
  Use when reviewing or writing an English math paper for style compliance.
- `references/ou-kk-style-alignment.md` — Concrete fix guide for 6 common
  Ou/KK style violations: explanatory parentheticals, roadmap paragraphs,
  flat case labeling, proof-structure meta-language, global condition
  declarations, and construction-section labels. Includes detection regex
  and before/after examples.

## Absorbed Skills

See `references/absorbed-skills.md` for the full historical record of
skills consolidated into this umbrella.

## CRITICAL: read_file / write_file Line-Number Pitfall

**When copying content from `read_file` output into `write_file`, you MUST strip
the line-number prefixes.**  `read_file` returns lines formatted as `LINE|CONTENT`;
if you pass this verbatim to `write_file`, the literal characters `1|`, `2|`, etc.
become visible in the compiled PDF (rendered as spurious `|` characters throughout).

**Correct workflow:**

1. Read the source with `read_file` to understand the content.
2. **Compose the translation in your head/scratchpad** — do NOT copy-paste
   `read_file` output directly into `write_file`.
3. Pass the clean (no prefix) content to `write_file`.
4. **After writing, compile and verify** that no artifacts appear:
   - Compile with `tectonic`
   - Check PDF spans for `|` characters (use `fitz` to search spans)
   - If found, strip with: `re.sub(r'^\d+\|', '', content, flags=re.MULTILINE)`

**Detection query** (run after compilation):
```python
import fitz
doc = fitz.open("main.pdf")
found = sum(1 for page in doc for b in page.get_text("dict")["blocks"]
            if "lines" in b for line in b["lines"]
            for span in line["spans"] if '|' in span["text"])
# found MUST be 0
```

## Reference: English Math Paper Writing Style

When writing or auditing an English mathematical paper (especially a
classification paper in the style of Kaygorodov--Khrypchenko), load
`references/math-paper-writing-style.md` for conventions on terminology,
condition declarations, and tutorial-voice detection.

## Writing Style: No Self-Coined Terms

Before translating, audit the English paper for self-coined terminology.
See `references/debranding-methodology.md` for the full methodology.

**Quick rule**: If KK and Ou don't name it, neither should we.
Replace named techniques with equation/lemma numbers,
index metaphors with direct math notation,
and capitalized observations with plain paragraph text.

## Translation Cross-Check Methodology

After translating all sections, run these checks (order matters):

> **Lean ↔ Paper audit** — after every round of paper edits, verify ALL
> mathematical claims against the Lean formalization. Full workflow at
> `references/lean-paper-audit.md`: extract Lean theorem signatures,
> build a paper-claim-to-Lean-theorem mapping table, verify condition
> alignment (char≠2, char≠3, n≥5), and confirm 0 sorries.

### 0. Full automated check (preferred)

Run `scripts/compliance-check.py` — a single Python script covering all
checks below (label parity, ref count, cite parity, punctuation, forbidden
words, spacing). See `scripts/compliance-check.py` for usage.

### 1. Label parity
```bash
diff <(grep -oh '\\\\label{[^}]*}' en/*.tex | sort) \
     <(grep -oh '\\\\label{[^}]*}' zh/*.tex | sort)
```
All labels must match exactly. Ignore comment-line references (`% ... \\ref{...}`).

**PITFALL — `\ref` count differs by 1?** Check for comment-line `\ref` before
concluding there's a missing reference. English source files often have section
separator comments like `% --- Main proof of Lemma~\ref{lem:X} ---` that the
Chinese version may lack. These don't appear in either PDF. Audit visible
(non-comment) refs only via `scripts/compliance-check.py` or manual inspection.

### 2. Reference count parity
```bash
grep -oh '\\eqref{[^}]*}\\|\\ref{[^}]*}' zh/*.tex | sort | uniq -c
```
Compare counts against English version. Every `\ref{lem:imageinI}` etc. must
appear the same number of times.

### 3. Cite parity
```bash
grep -oh '\\cite{[^}]*}' zh/*.tex | sort
```

### 4. Punctuation replacement
Chinese punctuation MUST be replaced with English equivalents:
`。→. ，→, ：→: ；→; （→( ）→)`
Do this OUTSIDE math mode only — split on `$`, replace in odd-indexed segments.
```python
segments = content.split('$')
for i in range(0, len(segments), 2):
    for cn, en in replacements.items():
        segments[i] = segments[i].replace(cn, en)
```

### 5. Forbidden word scan
```bash
grep -n '得到' zh-*.tex   # NEVER in body (§2+)
grep -n '迫使' zh-*.tex   # NEVER
grep -n '本文' zh-sec2.tex zh-*.tex  # NEVER in body
```

### 6. Impersonal tone
`grep -n '我们' zh-*.tex` — at most 0–1 occurrences. Replace with declarative.

### 7. First-occurrence English annotation
Key coined terms need parenthetical English on first use: `中心化的 (centered)`.
Standard terms (平凡导子, 换位子, 李代数) do not.

### 8. CJK ↔ math spacing
Chinese characters must be separated from `$...$` by a space:
`设 $F$ 为域` ✓ — `设$F$为域` ✗. Check both directions:
`[\u4e00-\u9fff]\$` (CJK touching `$`) and `\$[\u4e00-\u9fff]` (`$` touching CJK).

## Plan-Driven Translation: Regenerate When Source Changes

When the English paper has been revised after the translation plan was written,
**discard the old plan and regenerate from the current source.** An outdated
plan is worse than no plan — it references deleted sections, stale terminology,
and incorrect line counts. The 2026-07-12 translation required a complete
rewrite because 5 rounds of compression had reduced the source from ~792 to
610 lines and restructured multiple sections.

## Reference: Avoid Coined Technique Names

When writing or translating a classification-style math paper (KK/Ou/Filippov
tradition), do not coin formal names for proof techniques. Use lemma/equation
numbers instead. See `references/avoid-coined-terms.md` for the full analysis
with evidence from KK (2305.00727).

## LaTeX → Word Conversion

For converting ctex-based Chinese math papers to Word format, see
`references/latex-to-word-pandoc.md` for the full workflow. Key pitfalls:
- ctex commands (`\songti`, `\heiti`, `\kaishu`, `\zihao`) break pandoc
- `\input` sub-files must be manually resolved first
- Math survives as OMML (invisible to `.text` but rendered by Word)

## Word Format Extraction

When extracting format specifications from a `.docx` reference template,
see `references/docx-format-extraction.md` for the python-docx + XML
methodology. Covers page setup, font/size/bold per run, paragraph indentation,
line spacing, and Normal style baseline.

## ctex Formatting Tips

### 楷体加粗

`\kaishu\bfseries` does NOT produce visible bold in default ctex — 楷体 lacks
a real bold variant. Enable fake bold:

```latex
\setCJKfamilyfont{kaishu}{KaiTi}[AutoFakeBold={2.5}]
```

### 节编号紧贴标题

```latex
\ctexset{
  section = {
    name = {},
    number = \arabic{section},
    format = \heiti\zihao{4}\bfseries\raggedright,
    aftername = \hspace{0pt},
    beforeskip = 15.6pt plus .3pt minus .3pt,
    afterskip = 7.8pt plus .1pt minus .1pt,
  }
}
```

Produces "1预备知识" format (number directly attached, no space/period).

### 参考文献标题不加粗

ctex 的 `\section*` 继承节格式。若参考文献标题需黑体不加粗，临时覆盖：

```latex
\begingroup
\ctexset{section/format=\heiti\zihao{5}}
\input{zh-refs}
\endgroup
```

## Preliminaries: Docx-Style Restructuring

When matching a `.docx` reference template that uses no subsections, restructure
the preliminaries (zh-prelim.tex) as follows:

1. **Remove all `\subsection` headings.**  Docx-style theses typically flow as
   continuous prose with definitions and lemmas inline.

2. **Convert the "notation" subsection to a series of `\begin{definition}` blocks**
   for key concepts (ideal I, coordinate projection, etc.).  Keep lighter notation
   (matrix units, bracket formula) as flowing prose before the first definition.

3. **Keep lemmas and their proofs as `\begin{lemma}...\end{lemma}` blocks.**
   They become natural paragraph breaks in the continuous flow.

4. **Move "centered decomposition" into a `\begin{definition}`** rather than
   a standalone prose paragraph — it's a concept definition, aligning with
   the docx convention of `定义1.X`.

Resulting structure: prose → 定义1.1 → equation → 引理1.2 + proof → 引理1.3
+ proof → 定义1.2 → 引理1.4 + proof.  No subsection titles, continuous flow.

## Pitfall: Explanatory Parentheticals

Ou (2007) and KK (2023) use ZERO parentheticals that explain proof steps.
Parentheticals like `(using $d_3=d_1$ from above)`, `(only the bracket-right
term contributes, via...)`, or `(trivially if $u>i$, by Lemma~3.1)` are not
present in either reference. When auditing, note:

- **Automated regex scans are insufficient.**  Parentheticals span line breaks,
  use diverse keywords (`trivially`, `by`, `applied to`, `component ... cancels`),
  and hide in nested case analyses. A regex scan will find ~60% at best.
- **Manual line-by-line re-read of every proof is required** after the automated
  pass. Read each parenthetical and ask: "Would Ou write this explanation here?"
  If the answer is "no, he'd just state the result", delete it.
- **Check `\operatorname{char}F\neq3$` parentheticals too.** These are condition
  reminders that serve the same explanatory function.

See `references/blind-review-audit.md` for the full comparison methodology
against Ou and KK, including case-label nesting depth, roadmap paragraphs,
and pattern-language detection.

## Reference: Blind-Review Style Audit

When checking whether a paper reads like it belongs to the Ou/KK tradition,
audit these dimensions (methodology in `references/blind-review-audit.md`,
detailed fix examples and detection code in `references/ou-kk-style-alignment.md`):
1. **Roadmap paragraphs** — Ou has none. KK has ≤1 sentence. Any colon-enumerated
   "In Section 1 we... Section 2 constructs..." is a tell.
2. **Case-label nesting depth** — Ou uses at most (1)(2)(3). Three-level nesting
   (Case → condition → (i)(ii)(iii)) with bold labels and `\smallskip` reads like
   lecture notes.
3. **Explanatory parentheticals** — Ou/KK have 0. Any `(using/via/trivially/by
   Lemma/applied to)` parenthetical in a proof is a tell.
4. **Global hypothesis re-declaration** — "In what follows we work under the
   hypotheses of Theorem X. Thus char F≠2,3 and n≥5 throughout." Ou states
   conditions in the theorem and never repeats them.
5. **Pattern language** — "The same three-equation pattern as for u=1 applies"
   is meta-commentary on proof structure. Ou/KK never abstract proof steps
   into named patterns.

## Current Paper State (2026-07-12)

The canonical English paper is at `/workspace/paper-v3/`:
- **~508 lines, 5 pages, 0 errors**, 66 KB PDF
- elsarticle 3p, Times (newtx), tolerance=500
- All 8 theorems verified against Lean (0 sorries)
- **0 explanatory parentheticals, 0 coined terms, 0 metalanguage, 0 roadmap**
- §2 labels use (A)(B)(C) format matching Ou (2007)
- References: 5 entries, all verified (KK→Port. Math. 2024, Filippov+no.6, orphan lean removed)

Chinese translation: `/workspace/paper-zh/`
- ~492 lines, 6 pages, 0 errors, 198 KB PDF, ctexart 12pt
- Subsection headings left-aligned via `\ctexset`
- Lemma 3.1 (Claim) and Lemma 3.2 (im⊆I) reordered: Claim + proof first, then im⊆I + proof
- All `$1/2$` replaced with `$\frac{1}{2}$`

Formatted Chinese version: `/workspace/paper-zh-formatted/`
- ctexart 10.5pt (五号), 2cm margins, single spacing, 8 pages
- Format extracted from docx template: `FORMAT-SPEC.md`
- Includes Chinese+English title pages and abstracts
- 楷体 fake bold: `AutoFakeBold={2.5}`
- Pandoc-generated Word: `paper.docx` (22KB, 422 OMML formulas)

Lean formalization: `/opt/lean-home/lean-projects/e/E/Classification/` (11 files, 0 sorries)
Reference papers: `/workspace/参考论文/` (Ou 2007, KK 2023)

### Full debranding audit trail

See `references/debranding-audit.md` for Round 1 (named techniques) and Round 2 (index metaphors).
See `references/debranding-audit-r3.md` for Round 3 (parentheticals, metalanguage, case labels, roadmap, reference format).
See `references/reference-verification.md` for reference bibliography verification.
  - 0 self-coined technique names (chain bridge → \\eqref)
  - 0 roadmap paragraph
  - 0 nested bold case labels (flattened to If/If/If)
  - 0 proof-structure metalanguage
  - Global hypothesis: factual one-liner
- Chinese translation: `/workspace/paper-zh/`, ~492 lines, 6 pages, 197.6 KB PDF
  - All style fixes synced to Chinese
  - 0 forbidden words, 0 Chinese punctuation, 0 CJK↔$ spacing issues
- Translation plan: `/workspace/paper-zh/TRANSLATION-PLAN.md`

### Pre-translation Compression Workflow (5 rounds, 741→610 lines)

When preparing an English paper for translation, compress it first. Methodology:
1. **Benchmark**: compare against reference papers (Ou 2007, KK 2023) for density.
   Count words per page, display equations, and proof structure.
2. **Identify**: find display-math-for-single-equations, signpost sentences
   ("First, we prove..."), redundant restatements, and explanatory prose
   that readers can fill in themselves.
3. **Verify**: every compression must be cross-checked against Lean
   formalization to ensure no mathematical content is lost.
4. **Iterate**: multiple passes, each targeting a specific class of bloat
   (display→inline, signposts, bracket-expansion, construction verification).
5. **Check**: re-measure page count, PDF size, and line count after each round.

Full methodology and audit trail in `references/compression-methodology.md`.

### Translation Plan

The canonical translation plan for paper-v3 → paper-zh:
`/workspace/paper-zh/TRANSLATION-PLAN.md`

This plan includes:
- Corrected terminology table (all deleted terms removed)
- Section-by-section translation strategies aligned with current 610-line paper
- Lemma numbering verified against actual LaTeX `\label`/`\ref` structure
- ctexart preamble template

## Reference: Lean ↔ Paper Condition Audit

Before finalizing, verify that every lemma's conditions match the Lean
formalization. See `references/condition-audit-methodology.md` for the
full workflow: detect missing conditions, choose between per-lemma
conditions and global declarations, and verify post-fix.
