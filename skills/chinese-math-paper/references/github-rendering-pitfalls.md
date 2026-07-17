# GitHub Markdown Math Rendering — Reproduction Recipes

## Quick diagnostic

When math doesn't render on GitHub:
1. Check if it's in a **heading** (`#`, `##`, `###`) — GitHub never renders `$` in headings
2. Check if it's in a **markdown table** — `_` gets parsed as italic before MathJax
3. Check if `$` delimiters touch **CJK full-width** punctuation (`：`, `，`, `（`, `）`)
4. Check for `\operatorname` — replace with `\mathrm`

## Recipe 1: Math in markdown tables

**Symptom**: `$T_0 = Id$` renders as broken text with `_` triggering italic formatting.

**Root cause**: GitHub's markdown parser processes `_` as italic emphasis marker BEFORE MathJax sees the `$...$` region. The `_` in `T_0` or `E_{kk}` triggers italic parsing.

**Failed attempts**:
- `$T\_0$` — markdown strips the escape before MathJax, still broken
- `$ T_0 $` with spaces — does NOT help
- `\sb` (TeX subscript primitive) — NOT supported on GitHub MathJax

**Working fix**: HTML `<table>` with a blank line after the opening tag:

```html
<table>

<tr><td>$T_0 = \mathrm{Id}$</td></tr>
</table>
```

The blank line tells CommonMark to treat content as raw HTML (no markdown processing). The `_` passes through untouched to MathJax.

## Recipe 2: CJK punctuation blocking math

**Symptom**: `：$T(E_{kk})$，` — the `$` delimiters are not recognized and LaTeX displays as raw text.

**Root cause**: GitHub MathJax doesn't recognize `$` when it's immediately adjacent to full-width CJK characters (U+FF00-U+FFEF range).

**Working fix**: Add ASCII space between the CJK punctuation and `$`:

```
： $T(E_{kk})$ ，
```

## Recipe 3: `\operatorname` not supported

**Symptom**: `\operatorname{HalfDer}` displays as raw text.

**Working fix**: Use `\mathrm{HalfDer}` instead.

## Recipe 4: Math in headings

**Symptom**: `### Basis ($n+5$ dimensions)` shows literal `$n+5$`.

**Root cause**: GitHub does not invoke MathJax on heading text.

**Working fix**: Use plain text: `### Basis (n+5 dimensions)`.
