# Markdown + LaTeX Math Formatting — Pitfalls & Rules

Collected during PAPER-FINAL.md → PAPER-FINAL-zh.md formatting (2026-06-20).

---

## Core Rule: Plain `$$` Only

**Do NOT nest `\begin{aligned}` or `\begin{align}` inside `$$` blocks.**

`\tag{...}` is an amsmath top-level command. It does NOT work inside
`aligned` (an inner environment) and has unreliable behavior inside `align`
when wrapped in `$$`. The only reliably correct pattern is:

```markdown
$$
formula here \tag{1.1}
$$
```

One equation per `$$` block. One `\tag` per block.

---

## Multi-Equation Blocks → Split

**Before (WRONG):**
```markdown
\begin{align}
A &= B \tag{1} \\[4pt]
C &= D \tag{2}
\end{align}
```

**After (CORRECT):**
```markdown
$$
A = B \tag{1}
$$

$$
C = D \tag{2}
$$
```

- `&` alignment markers: remove entirely.
- `\\[4pt]` spacing between equations: replace with blank line + new `$$` block.
- `\\` line continuations within a single equation: keep, but use only `\\` (no `[dim]`).

---

## Line Breaks within a Single Multi-Line Equation

For equations too long for one line, use `\\` inside `$$`:

```markdown
$$
2 \cdot c_{ij}^{uv} = (\text{first term}) \\
\qquad - (\text{second term}) \\
\qquad + (\text{third term}). \tag{2.7}
$$
```

- `\\` works for line breaks in plain `$$` (MathJax and most renderers).
- `\\[4pt]`, `\\[2pt]`: do NOT use in plain `$$`. Only works inside `aligned`/`align`.
- Use `\qquad` for indentation of continuation lines.

---

## Chinese Text: No Line Wrapping

Chinese paragraphs should be on **single lines**. The ~80-character line wrapping
inherited from English originals creates unnecessary mid-sentence breaks that
look wrong in the source.

**Before (WRONG — inherited English wrapping):**
```markdown
每个 1/2-导子唯一地分解为 D = λ·Id + T，其中 λ ∈ F 且 T 的像包含在
中心 Z(N_n) = span{E_{1n}} 中。等价地，当 j−i ≥ 2 时 T(E_{ij}) = 0，
而每个相邻基元素 E_{i,i+1} 可以任意映射到 Z(N_n) 中。
```

**After (CORRECT — joined paragraph):**
```markdown
每个 1/2-导子唯一地分解为 D = λ·Id + T，其中 λ ∈ F 且 T 的像包含在中心 Z(N_n) = span{E_{1n}} 中。等价地，当 j−i ≥ 2 时 T(E_{ij}) = 0，而每个相邻基元素 E_{i,i+1} 可以任意映射到 Z(N_n) 中。
```

In markdown, single newlines within paragraphs are treated as spaces during
rendering, so mid-sentence breaks don't affect output. But they make the
source file messy and harder to edit. Join all paragraph lines.

---

## Consecutive `$$` Markers

Two consecutive `$$` on adjacent lines (closing one block and opening the next
with no blank line between) creates rendering ambiguity. Always separate `$$`
blocks with blank lines:

**WRONG:**
```markdown
$$
first equation
$$
$$
second equation
$$
```

**CORRECT:**
```markdown
$$
first equation
$$

$$
second equation
$$
```

---

## Validation Checklist

When finalizing a paper in markdown+LaTeX:

1. `$$` markers: count must be even (each open has a close).
2. `$` markers: count must be even per paragraph outside display math.
3. `\tag`: every instance must be inside a `$$` block.
4. `\begin{aligned}`, `\begin{align}`: count must be **zero**.
5. Consecutive `$$`: none (two `$$` on adjacent lines = error).
6. Heading hierarchy: no jumps (h2→h4, etc.).
7. All sections present.
8. Chinese paragraphs: no mid-sentence line breaks.

A Python validation script can automate checks 1-7. See the check script
used in the N_n paper session for reference.

---

## GitHub README Rendering (2026-07-07)

When markdown is rendered on GitHub (README.md, etc.), MathJax has additional
restrictions beyond standard LaTeX-in-markdown:

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| `\operatorname{X}` | "not allowed" warning badge on README | Use `\mathrm{X}` |
| `$x$` inside headings (`##`, `###`) | Literal dollar signs displayed | Remove math from headings; use plain `x` |
| `$T_0$` in markdown table cells | Text breaks at `_` — italic parsing kicks in | Use `$T\sb{0}$` instead of `$T_0$` |
| `$E_{kk}$` in HTML `<table>` cells | **Same breakage** — GitHub parses markdown *inside* HTML | Use `$E\sb{kk}$` everywhere in tables |
| `\sb` double-escaped by patch tool | File has `\\sb` (two backslashes) | Use `write_file` or post-fix with sed |

**The `\sb` rule**: In ANY GitHub table cell (markdown or HTML), use `\sb{...}` 
instead of `_{...}` for subscripts. `\sb` is the TeX primitive — MathJax renders 
it identically, but it contains no `_` for markdown to misinterpret.

**Escape verification**: `python3 -c "print(hex(open('f').read()[open('f','rb').read().find(b'sb')-1]))"` — must return `0x5c`.
