# GitHub README Math Rendering — Complete Rulebook

> These rules are hard-won from trial and error on `wu1chenghui/half-derivations-nilpotent-lie-algebra`.
> GitHub uses MathJax for `$...$` and `$$...$$`, but the markdown parser runs FIRST and can corrupt LaTeX before MathJax sees it.

## Rule 1: `\operatorname` → `\mathrm`

GitHub MathJax does not support `\operatorname`. Replace everywhere:

```
❌ $\operatorname{HalfDer}(N_n)$
✅ $\mathrm{HalfDer}(N_n)$
```

## Rule 2: No `$...$` in headings

Headings are markdown-only; MathJax does not process them.

```
❌ ### Basis ($n+5$ dimensions)
✅ ### Basis (n+5 dimensions)
```

## Rule 3: Tables with math → use HTML `<table>` with blank line

Markdown tables parse `_` as italic BEFORE MathJax. This breaks all subscripts.
The fix: use an HTML `<table>` with a blank line after `<table>` to block markdown processing.

```html
### Basis

<table>

<tr><th>Type</th><th>Description</th></tr>
<tr><td><b>Id</b></td><td>$T_0 = \mathrm{Id}$</td></tr>
<tr><td><b>A_k</b></td><td>$T(E_{kk}) = E_{1,n}$</td></tr>
</table>
```

Key: the blank line between `<table>` and `<tr>` is essential. Without it, markdown still processes `_` inside the HTML.

## Rule 4: CJK full-width punctuation needs ASCII space around `$`

Full-width characters (`：` `，` `（` `）`) touching `$` delimiters prevent MathJax from recognizing the math region.

```
❌ 中心映射：$T(E_{kk}) = E_{1,n}$，其余为零
✅ 中心映射： $T(E_{kk}) = E_{1,n}$ ，其余为零
```

## Rule 5: `\sb` (TeX subscript primitive) does NOT work on GitHub MathJax

Do not use `\sb{0}` as a workaround for underscores. It fails silently.

```
❌ $T\sb{0} = \mathrm{Id}$   — not rendered
✅ $T_0 = \mathrm{Id}$        — works in HTML tables
```

## Rule 6: `\tag` in `$$` display math → use plain `$$...$$`

Markdown `$$` blocks should not contain `\tag`. Use plain display math.

## Quick Checklist

- [ ] All `\operatorname` → `\mathrm`
- [ ] No `$` in headings
- [ ] Tables: HTML `<table>` + blank line
- [ ] CJK near `$`: ASCII spaces added
- [ ] No `\sb` usage
- [ ] Verify on actual GitHub page (rendering differs from local preview)
