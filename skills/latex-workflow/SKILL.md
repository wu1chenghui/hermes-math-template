---
name: latex-workflow
description: Markdown→LaTeX→PDF pipeline — pandoc conversion, Unicode math handling, tectonic/xelatex compilation, and common pitfalls.
---

# Markdown to LaTeX Pipeline

Convert mathematical papers and documents from Markdown to production-quality PDF via pandoc + tectonic.

## Trigger Conditions
- User asks to produce LaTeX or PDF from markdown/text
- User needs LaTeX tooling installed or configured
- Mathematical papers with Unicode math symbols that need conversion
- Compiling .tex files to PDF

## Installation (non-root / static binaries)

When apt is unavailable (no root, Docker container, etc.), use static binaries:

```bash
# Tectonic (LaTeX engine, ~20MB)
curl -fsSLO https://github.com/tectonic-typesetting/tectonic/releases/download/tectonic%400.15.0/tectonic-0.15.0-x86_64-unknown-linux-gnu.tar.gz
tar xzf tectonic-*.tar.gz && mv tectonic ~/.local/bin/

# Pandoc (document converter, ~50MB)
curl -fsSLO https://github.com/jgm/pandoc/releases/download/3.6.4/pandoc-3.6.4-linux-amd64.tar.gz
tar xzf pandoc-*-linux-amd64.tar.gz && cp pandoc-*/bin/pandoc ~/.local/bin/

# Ensure PATH includes ~/.local/bin
export PATH="$HOME/.local/bin:$PATH"
```

Tectonic auto-downloads missing TeX packages on first compile — no need for texlive-full.

## Pipeline Overview

```
Input.md
  → (1) Convert ``` code blocks → $$ display math
  → (2) Convert Unicode math symbols → LaTeX commands
  → (3) pandoc --to latex --standalone (with custom preamble)
  → (4) tectonic → Output.pdf
```

## Critical Pitfall: Unicode → LaTeX Command Separation

When converting Unicode math symbols to LaTeX commands, every command that could be followed by a letter MUST end with `{}`. TeX parses `\commandName` by consuming all consecutive letters — so `\cdotD` becomes undefined command `\cdotD`, not `\cdot` followed by `D`.

**Every one of these needs `{}`:**

- Operators: `\cdot{}`, `\times{}`
- Relations: `\le{}`, `\ge{}`, `\ne{}`
- Arrows: `\to{}`, `\in{}`
- Logic: `\land{}`
- ALL Greek letters: `\lambda{}`, `\delta{}`, `\Sigma{}`, etc.

Correct mapping example — see `references/unicode-map.md` for the full table.

**DO NOT** use:
```python
"·": r"\cdot",      # WRONG — 2·D → 2\cdotD (undefined)
"λ": r"\lambda",    # WRONG — λId → \lambdaId (undefined)
```

**DO use:**
```python
"·": r"\cdot{}",    # 2·D → 2\cdot{}D
"λ": r"\lambda{}",  # λId → \lambda{}Id
```

Subscript digits (₁,₂,...) convert to `_{1}`, `_{2}` — the braces are structural here and don't need the extra `{}`.

The conversion must run AFTER code-block→$$ conversion so math blocks get Unicode-converted too.

## Preamble Template

See `references/preamble.tex` for an English starter preamble. For Chinese papers, use `references/preamble-zh.tex` (ctex-based, CJK fonts, Chinese theorem names).

## Chinese Paper Support (ctex)

For papers mixing Chinese prose with LaTeX math, use `ctexart` + a CJK font:

```bash
pandoc input-zh.md \
  -f markdown -t latex -o output.tex --standalone \
  --metadata documentclass=ctexart \
  --metadata classoption="UTF8,12pt,a4paper" \
  --include-in-header preamble-zh.tex
```

Key preamble differences from English:
- **No `\usepackage[T1]{fontenc}` or `\usepackage{lmodern}`** — these switch the text font to Latin Modern, which lacks CJK glyphs AND Unicode math symbols. Let ctex (fontspec) handle all font selection.
- `\setCJKmainfont{WenQuanYi Zen Hei}` — use any available CJK font
- Theorem env names in Chinese: `定理`, `命题`, `引理`, `推论`

See `references/preamble-zh.tex` for a complete starter preamble.

Tectonic auto-downloads `ctex`, `xeCJK`, and Fandol fallback fonts. If a system CJK font is available (check with `fc-list :lang=zh`), prefer it over Fandol for better glyph coverage.

## Fixing Over-Wide Equations

Display equations with long `\\text{}` blocks (especially Chinese) can exceed page width.

### English Math Paper Overfull Fixes

See `references/overfull-hbox-fixes.md` for 7 targeted patterns: inline→display,
split concatenated displays, sentence breaking, paragraph reflow, `\\sloppy`,
URL line breaking, and amsart title block quirks.

### Chinese-Specific Fixes

**Pitfall: `\tag` in `aligned`**
`\tag{...}` only works in top-level amsmath environments (`align`, `equation`, `gather`), NOT in `aligned` inside `\[...\]`. Use raw `\begin{align}...\end{align}` directly (pandoc will pass it through).

**Pitfall: Pandoc escapes `{` `}`**
Markdown `{...}` groups get escaped to `\{...\}` by pandoc. When you need a scope group, put `\small` (or other size commands) INSIDE the environment:
```latex
\begin{align}
\small                    % ← inside, not outside
2 \cdot c = &\ ... \nonumber \\
            &- ... \tag{2.7}
\end{align}
```
Do NOT wrap with `{\small ... }` — pandoc will break it.

**Pattern: `align` + `\small` for multi-line conditional equations**
```latex
\begin{align}
\small
expr = &\ (\text{若 } P_1 \text{ 则 } X_1 \text{ 否则 } 0) \nonumber \\
       &- (\text{若 } P_2 \text{ 则 } X_2 \text{ 否则 } 0) \nonumber \\
       &+ (\text{若 } P_3 \text{ 则 } X_3 \text{ 否则 } 0) \nonumber \\
       &- (\text{若 } P_4 \text{ 则 } X_4 \text{ 否则 } 0). \tag{2.7}
\end{align}
```
`\nonumber` on all but the last line. `\small` scopes automatically to the environment.

## Thesis Template (University-Specific Formatting)

When the user needs a formatted thesis (not a journal article), the approach differs from the standard pipeline. Instead of letting pandoc generate the full document, generate a body fragment and stitch it into a pre-built template.

### Pipeline: Markdown → Thesis .tex

```
PAPER.md
  → (1) Shift heading levels: ##→#, ###→##, ####→###  (so pandoc maps to chapter/section)
  → (2) Remove top-level # Title (handled by template)
  → (3) pandoc --top-level-division=chapter → body fragment (no --standalone)
  → (4) Stitch body into template at insertion point
  → (5) tectonic → thesis.pdf
```

### FJNU Template (福建师范大学本科论文)

Reference: `references/fjnu-thesis-format.md` — complete formatting spec extracted from the university's official page.

Key specs:
- A4, margins: left=2.5cm(含0.5cm装订线), right=2cm, top=2cm, bottom=2cm
- 单倍行距, 首行缩进2汉字
- 章=四号黑体顶格, 节=小四黑体, 正文=五号宋体
- 章节编号: 1 / 1.1 / 1.1.1 (理工科)
- 公式按章编号: (2.1), (2.2)...
- 定理环境中文化

Template file: `templates/fjnu-thesis-template.tex` — complete standalone LaTeX document with:
- ctexrep document class
- All formatting preset via `\ctexset`
- Placeholder commands for author info, abstract, keywords
- Built-in cover page, abstract, TOC, references environment
- Pandoc compatibility shims (tightlist, longtable, booktabs, array, calc)

### Heading Level Shift (Critical)

The paper uses `## Section` for top-level sections. Pandoc maps `#`→section, `##`→subsection in article mode. For thesis with chapters:
1. Shift headings UP one level: `##`→`#`, `###`→`##`, `####`→`###`
2. Use `pandoc --top-level-division=chapter` so `#`→`\chapter`, `##`→`\section`
3. **Do NOT add `#` to the front** — `"#" + "##"` = `"###"` which is the WRONG direction. Use `line[1:]` to remove one `#`.

### Stitching Body into Template

The template file must have a marker where body content is inserted. The FJNU template uses:
```latex
%── 正文 ──
```
Everything before this is preamble + cover + abstract + TOC. Body goes between the marker and `\end{document}`.

### Pandoc Compatibility Packages

When using a custom template (not pandoc's default), these packages are often missing
and cause "undefined environment" errors. Always include in the template:

```latex
\providecommand{\tightlist}{%                   % pandoc's compact list spacing
  \setlength{\itemsep}{0pt}\setlength{\parskip}{0pt}}
\usepackage{longtable,booktabs,array,multirow,calc}  % pandoc tables
\usepackage{tabularx}
\usepackage{xcolor}
```

Without these, errors like `Undefined control sequence \tightlist` or
`Environment longtable undefined` occur silently in the PDF build.

## Verification
```bash
tectonic paper.tex          # single run, handles cross-refs internally
# Check for warnings:
tectonic paper.tex 2>&1 | grep -cE 'warning|Missing|Overfull'
# 0 = clean
```

## Absorbed Skills

The following skills have been consolidated into this one:

### `latex-compilation` (document-production/)
- Content: Markdown→LaTeX→PDF via pandoc + tectonic, Chinese/thesis formatting.
- All techniques and pitfalls are covered above. Additional unique reference
  (`references/pipeline-scripts.md`, project-specific paths) was not migrated.
- Thesis format spec preserved as `references/fjnu-thesis-format.md`.

### `latex-md-pipeline` (productivity/)
- Content: Same English + Chinese pipeline with project-specific script paths.
- Unicode mapping table preserved as `references/unicode-map.md`.
- **Preamble templates absorbed here:**
  - `templates/preamble-en.tex` — English paper preamble (amsmath, amsthm, lmodern)
  - `templates/preamble-zh.tex` — Chinese paper preamble (ctex, WenQuanYi Zen Hei)
  Use them as starter files for new papers.
