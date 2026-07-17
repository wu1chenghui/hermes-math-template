# Chinese Mathematical Paper Formatting

Conventions for writing mathematical papers in Chinese, for LaTeX with `ctex`.

## Document Class

Use `ctexart` for standard Chinese math papers:

```tex
\documentclass[12pt,a4paper]{ctexart}
```

`ctex` handles Chinese font selection automatically (Song/Ming for body, Hei for headings). Compatible with `tectonic`.

## Theorem Environments (Chinese Names)

```tex
\usepackage{amsthm}
\newtheorem{theorem}{定理}[section]
\newtheorem{lemma}[theorem]{引理}
\newtheorem{corollary}[theorem]{推论}
\newtheorem{proposition}[theorem]{命题}
\theoremstyle{definition}
\newtheorem{definition}[theorem]{定义}
\theoremstyle{remark}
\newtheorem{remark}[theorem]{注}
```

The `amsthm` package auto-generates "证明" for the `proof` environment when `ctex` is loaded.

## Punctuation: USE ENGLISH, NOT CHINESE

**CRITICAL**: Chinese math papers overwhelmingly use English (Western) punctuation, not Chinese full-width punctuation. The reason: Chinese period "。" (U+3002) looks identical to subscript "₀" in many fonts, creating confusion in formula-dense papers.

| Use | NOT |
|-----|-----|
| `.` (英文句点) | `。` (中文句号) |
| `,` (英文逗号) | `，` (中文逗号) |
| `:` (英文冒号) | `：` (中文冒号) |
| `;` (英文分号) | `；` (中文分号) |
| `()` (半角括号) | `（）` (全角括号) |

This is LaTeX's default behavior — no special configuration needed.

## Section Titles (Chinese)

| English | Chinese |
|---------|---------|
| Introduction / §1 | 引言 / §1 |
| Preliminaries | 预备知识 |
| Main Results | 主要结果 |
| Proof | 证明 |
| References | 参考文献 |
| Abstract | 摘要 |
| Keywords | 关键词 |

## Translation Conventions

- **Math formulas**: NEVER translate. Keep exactly as in English version.
- **LaTeX commands**: NEVER translate. `\ref`, `\eqref`, `\label` all preserved.
- **Prose/narrative text**: Translate to Chinese.
- **References**: Keep in English (bibliography entries are untranslated).
- **Symbols**: `N(n,F)`, `Δ`, `E_{ij}`, `φ`, etc. all kept in Latin script.
- **Notation like "1/2-derivation"**: Translate as "1/2-导子".

## Files Structure

Typical multi-file project for Chinese translation:

```
paper-zh/
├── main.tex              ← Chinese config + preamble
├── zh-abstract.tex
├── zh-intro.tex
├── zh-prelim.tex
├── zh-sec2.tex
├── zh-strategy.tex
├── zh-image-restriction.tex
├── zh-endpoint.tex
└── zh-refs.tex           ← Untranslated (English)
```

## Sample main.tex

```tex
\documentclass[12pt,a4paper]{ctexart}
\usepackage{amsmath,amssymb,amsthm,amscd,mathtools}
\usepackage{microtype}
\usepackage[hyphens]{url}

\newtheorem{theorem}{定理}[section]
\newtheorem{lemma}[theorem]{引理}
\newtheorem{corollary}[theorem]{推论}
\newtheorem{proposition}[theorem]{命题}
\theoremstyle{definition}
\newtheorem{definition}[theorem]{定义}
\theoremstyle{remark}
\newtheorem{remark}[theorem]{注}

\DeclareMathOperator{\ad}{ad}
\DeclareMathOperator{\im}{im}
\DeclareMathOperator{\id}{id}

\title{N(n,F)的1/2-导子分类}
\author{作者}
\date{}

\begin{document}
\maketitle

\begin{abstract}
[中文摘要...]
\end{abstract}

\input{zh-intro}
\input{zh-prelim}
\input{zh-sec2}
\input{zh-strategy}
\input{zh-image-restriction}
\input{zh-endpoint}

\begin{thebibliography}{99}
% English references, unchanged
\end{thebibliography}
\end{document}
```

## Specific Journal Requirements (from web research 2026-07-11)

### 数学年刊 A辑 (Chinese Annals of Mathematics, Series A)

Source: https://camath.fudan.edu.cn/camacn/ch/index.aspx → 征稿简则

- **Format**: A4 paper, double-spaced (两倍行距)
- **Abstract**: ≤200 Chinese characters + English translation
- **Keywords**: 3–8 + 2000 MR Subject Classification + CLC number
- **Author**: Family name first, given name last
- **Source files**: CCT format (not LaTeX/ctex directly; CCT is the older Chinese TeX system)
- **References**: Chinese GB/T format, numbered by appearance order
- **Title/Abstract/Keywords**: Must provide BOTH Chinese and English versions
- **Template**: "AC辑中文模板" available for download from journal website (requires login)

### 数学学报中文版 (Acta Mathematica Sinica, Chinese Series)

Source: https://actamath.cjoe.ac.cn/Jwk_sxxb_cn/ (403 blocked)

- Uses custom `amse.cls` document class
- Chinese + English abstracts required
- LaTeX or CCT source files accepted

### 应用数学学报 (Acta Mathematicae Applicatae Sinica)

- Chinese version uses CCT format
- English version uses standard LaTeX

### General Observations

- Most Chinese math journals still use CCT (Chinese TeX) rather than modern ctex/LaTeX
- For self-translation (not targeting a specific journal), `ctexart` is the recommended modern approach
- The double-spacing requirement (两倍行距) is common for initial submissions
- All journals require both Chinese and English title/abstract/keywords
