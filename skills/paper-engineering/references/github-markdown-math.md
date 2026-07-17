# GitHub Markdown Math Rendering — Pitfalls & Fixes

When publishing a mathematical paper's README on GitHub, four rendering traps
cause `$...$` LaTeX to display as raw text or break entirely. These are
GitHub-platform-specific and affect any math-heavy repository.

## 1. `\operatorname` not supported

**Symptom**: Formula displays as raw text with unrendered `\operatorname`.

**Fix**: Replace ALL `\operatorname{...}` with `\mathrm{...}`.

```diff
- $\dim \operatorname{HalfDer}(N_n) = n+5$
+ $\dim \mathrm{HalfDer}(N_n) = n+5$
```

## 2. `$...$` in headings not rendered

**Symptom**: Heading like `### Basis ($n+5$ dimensions)` shows literal `$n+5$`.

**Fix**: Use plain text in headings. MathJax does not process `$...$` inside
Markdown headings (`#`, `##`, `###`, etc.).

```diff
- ### Basis ($n+5$ dimensions)
+ ### Basis (n+5 dimensions)
```

## 3. Underscores in table cells break LaTeX

**Symptom**: `$T_0$` in a Markdown table cell renders as separate characters
(`T`, `0`) because `_` is parsed as italic emphasis by the Markdown engine
BEFORE MathJax sees the `$...$` region. Adding `\` before `_` (`$T\_0$`)
does NOT fix this — GitHub strips the escape before MathJax processes it.
Using `\sb{...}` (TeX subscript primitive) also fails — not supported by
GitHub's MathJax configuration.

**Fix**: Use raw HTML `<table>` with a blank line after the opening tag.
The blank line tells CommonMark to treat content as raw HTML, preventing
underscore parsing:

```html
### Table Title

<table>

<tr><th>Header</th><th>Header</th></tr>
<tr><td>Value</td><td>$T_0 = \mathrm{Id}$</td></tr>
<tr><td>Value</td><td>$T(E_{kk}) = E_{1,n}$</td></tr>
</table>
```

Key rules for this to work:
- `<table>` on its own line, followed by a BLANK line, then `<tr>` tags
- No markdown table syntax (`|...|`) — pure HTML only
- Underscores inside `$...$` are safe because Markdown parser skips the
  HTML block content

## 4. Full-width CJK punctuation touching `$` prevents MathJax

**Symptom**: `$T(E_{kk}) = E_{1,n}$` renders correctly in an English file
but fails in a Chinese file with the same HTML structure.

**Root cause**: Full-width punctuation characters (U+FF1A `：`, U+FF0C `，`,
U+FF08 `（`, U+FF09 `）`) touching the `$` delimiters prevent MathJax from
recognizing the math region boundary.

**Fix**: Add ASCII spaces between full-width CJK punctuation and `$`
delimiters:

```diff
- 中心映射（center maps）：$T(E_{kk}) = E_{1,n}$，其余为零
+ 中心映射（center maps）： $T(E_{kk}) = E_{1,n}$ ，其余为零
```

The space before `$` and after `$` separates the CJK punctuation from the
math delimiter without affecting the rendered math appearance.

## Verification checklist

After making fixes, check on the actual GitHub rendered page (not preview):
- [ ] All `\operatorname` replaced with `\mathrm`
- [ ] No `$...$` in any `#` / `##` / `###` headings
- [ ] All tables with math use HTML `<table>` + blank line
- [ ] CJK text adjacent to `$` has ASCII spaces
- [ ] Force-refresh (Ctrl+Shift+R) to bypass browser cache
