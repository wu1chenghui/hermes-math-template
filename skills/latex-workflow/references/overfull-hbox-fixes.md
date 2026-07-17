# Overfull Hbox Fixes for Math Papers

Patterns that resolve overfull hbox warnings in amsart/tectonic LaTeX math papers.

## Pattern 1: Inline Math → Display Math

When a single inline formula is too wide:
```
BAD:  $\pi_{k+1,k+3}(\varphi(E_{k+1,k+3}))=\frac12(d_{k+1}+d_{k+2})$.
FIX:  \[ \pi_{k+1,k+3}(\varphi(E_{k+1,k+3}))=\tfrac12(d_{k+1}+d_{k+2}). \]
```

## Pattern 2: Split Concatenated Display Formulas

When two formulas are joined with `\qquad`:
```
BAD:  \[ A = B, \qquad C = D. \]
FIX:  \[ A = B \] and \[ C = D. \]
```
Use `\tfrac` instead of `\frac` in displays to save width.

## Pattern 3: Long Inline Math → Break Into Sentences

```
BAD:  Thus the only potentially non-zero $a$-channel variables are $a_1,a_{n-2},a_{n-1}$, and the only $c$-channel variables are $c_1,c_2,c_{n-1}$.
FIX:  Thus the only $a$-channel variables that can be non-zero are $a_1$, $a_{n-2}$, $a_{n-1}$. The only $c$-channel variables that can be non-zero are $c_1$, $c_2$, $c_{n-1}$.
```
Key technique: separate each variable into its own `$...$` to give TeX line-breaking points.

## Pattern 4: Paragraph Reflow

For paragraphs that are just barely overfull (2-15pt), slightly different line breaks often fix it. Change "for all" → "for every", adjust where words land on line boundaries, or shorten a phrase ("explicit construction of" → "constructing").

## Pattern 5: `\\sloppy` is a TRAP — DO NOT USE

**`\\sloppy` does NOT fix overfull hboxes.** It converts them into underfull
hboxes by stretching inter-word spaces. The result is **visibly ugly uneven
spacing** throughout the document. The problem is hidden (warnings change from
overfull to underfull) but the visual quality degrades.

Instead, use the balanced approach:
```tex
\\usepackage{microtype}
\\tolerance=500            % default 200 — allow moderate flexibility
\\emergencystretch=0.5em   % extra stretch only on hardest lines
```

This keeps tight spacing on easy lines while allowing slightly looser breaks
only where needed. Together with microtype (character protrusion + font
expansion), this eliminates ~60% of overfull warnings without the ugly
side effects of `\\sloppy`.

## Pattern 6: Diagnostic Methodology

When facing multiple overfull warnings, DON'T guess at fixes. Systematically test:

1. **Establish baseline**: count overfull/underfull in current config
2. **Test `\\sloppy` removal**: remove if present; observe real overfull count
3. **Add microtype** (`\\usepackage{microtype}`): remeasure
4. **Add balanced tolerance**: `\\tolerance=500`, `\\emergencystretch=0.5em`
5. **Test font size**: compile at 10pt vs 12pt (**NOTE**: amsart textwidth is
   identical at both sizes — 360pt = 5in — so changing font size doesn't change
   what fits per line)
6. **Test Times fonts** (`\\usepackage{newtxtext,newtxmath}`): Times is narrower
   than Computer Modern in some contexts, wider in others — always test before
   adopting

After these systematic tests, fix remaining overfulls (those >10pt) with
Patterns 1-4 below.

## Pattern 7: URL Line Breaking

```
BAD:  \verb|https://github.com/leanprover-community/mathlib4|
FIX:  \usepackage[hyphens]{url}  ...  \url{https://github.com/leanprover-community/mathlib4}
```
The `[hyphens]` option allows URLs to break at hyphens.

## Pattern 8: amsart Title Block Overfull

The amsart `\maketitle` creates an unbreakable title box. If the title contains inline math (e.g., `$1/2$`), the entire title becomes unbreakable. This is usually harmless — the overfull is in the title area, not body text. Mitigations: use a shorter running head with `\title[short]{long}`, or remove empty `\author{}`/`\date{}` commands that may create spurious boxes.
