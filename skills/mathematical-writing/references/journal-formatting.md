# Journal Formatting Requirements for Mathematics Papers

## Two Major Publishing Systems

| | **AMS** | **Elsevier** |
|---|---|---|
| Representative journals | Trans. AMS, Proc. AMS, JAMS | LAA, J. Algebra, Adv. Math |
| Document class | `amsart.cls` | `elsarticle.cls` |
| Default font size | 10pt | 9–10pt (Times) |
| Default font | Computer Modern | Times (Times New Roman) |
| Initial submission | Any reasonable format accepted | Any reasonable format accepted |
| Final typesetting | AMS does it themselves | Author uses elsarticle |

## amsart Page Geometry (EMPIRICALLY VERIFIED — 2026-07-11)

Compiled with tectonic, `\typeout`-based measurement:

| Parameter | 10pt amsart | 12pt amsart |
|-----------|:-----------:|:-----------:|
| textwidth | 360.0pt (5in) | 360.0pt (5in) |
| textheight | 584.0pt | 584.0pt |
| oddsidemargin | 54.88pt | 54.88pt |
| topmargin | 22.22pt | 22.22pt |
| baselineskip | 12.0pt | 14.0pt |
| parindent | 12.0pt | 12.0pt |

**Key finding**: amsart uses THE SAME text block for 10pt and 12pt.
The only difference is baselineskip (12pt vs 14pt). This means:

- At 10pt: ~78 chars/line, ~49 lines/page — slightly above ideal reading width
- At 12pt: ~65 chars/line, ~42 lines/page — closer to ideal 60-70 chars/line

**Recommendation**: 12pt is NOT a problem for amsart. The text block is fixed;
12pt gives better readability (fewer chars/line). Do NOT recommend switching to
10pt as a blanket fix — measure first.

## The `\sloppy` Trap

**`\sloppy` (tolerance=9999) is a BAND-AID that makes typesetting WORSE.**

It converts overfull hboxes (lines exceeding the margin) into underfull hboxes
(stretched spacing within the margin). The result is uneven word spacing that
looks unprofessional. Professional math papers NEVER use `\sloppy`.

### Empirical test results (our 834-line paper, 2026-07-11)

| Configuration | Hbox warnings | Nature |
|--------------|:---:|--------|
| 12pt CM + `\sloppy` (original) | 8 | All underfull (spacing looks bad) |
| 12pt CM + microtype (no sloppy) | 15 | 14 overfull + 1 underfull |
| 10pt CM + microtype (no sloppy) | 10 | All overfull |
| 10pt Times + microtype (no sloppy) | 20 | Worse — Times fonts increase overflow |
| 12pt CM + microtype + tolerance=500 | 8 | 6 overfull + 2 underfull |

### The correct fix (in priority order)

1. **Remove `\sloppy`** — stop hiding the real problem.
2. **Add `\usepackage{microtype}`** — subtle character protrusion/font expansion; reduces some overfulls.
3. **Use balanced tolerance** (NOT `\sloppy`'s 9999):
   ```tex
   \tolerance=500
   \emergencystretch=0.5em
   ```
   This allows slight extra stretch on genuinely hard lines while keeping
   easy lines tight. Overfulls < 5pt are invisible in print.
4. **Fix the worst individual overfull lines** by rewording or splitting
   into display math. After steps 1–3, only ~2–3 lines will remain problematic.

### Times fonts do NOT reduce overfulls on amsart

Contrary to common assumption, switching to Times (newtxtext + newtxmath)
increased overfull count from 10 to 20 in our tests. Times metrics interact
poorly with amsart's 360pt textwidth. Only switch fonts when submitting to
an Elsevier journal that requires elsarticle.cls.

### Font size: 10pt vs 12pt

Since amsart textwidth is identical for 10pt and 12pt (360pt), the choice
affects only chars/line and pages. 12pt actually yields the ideal 65 chars/line
for mathematical text. Do NOT reflexively switch to 10pt — it's not the
solution to overfull boxes.

## Quick Typesetting Quality Audit

To diagnose a paper's formatting health:

```bash
# Compile and count hbox warnings
tectonic main.tex 2>&1 | grep -c "Overfull\|Underfull"

# Check which sections have the worst problems
tectonic main.tex 2>&1 | grep "Overfull\|Underfull" | sed 's/:.*//' | sort | uniq -c | sort -rn
```

Goal: ≤ 1–2 warnings per output page. If more, check the worst sections.

## Journal-Specific Notes

### LAA (Linear Algebra and its Applications) — Elsevier
- Uses `elsarticle.cls`. Accepts `amsart` for initial submission.
- Ou-Wang-Yao (2007) published here.
- Times font (newtxtext/newtxmath) for final CRC.

### Journal of Algebra — Elsevier
- Same as LAA: `elsarticle.cls`, accepts any format for review.

### Transactions of the AMS
- Requires AMS author package (journal-specific template).
- `amsart.cls` with strict top-matter rules.

## elsarticle Page Geometry (EMPIRICALLY VERIFIED — 2026-07-11)

Compiled with tectonic + newtxtext/newtxmath, `\typeout`-based measurement:

| Parameter | 1p (preprint) | 3p (final single-col) | 5p (final two-col) |
|-----------|:---:|:---:|:---:|
| textwidth | 384pt (5.33in) | **468pt** (6.50in) | 522pt (7.25in) |
| textheight | 562pt | 622pt | 682pt |
| oddsidemargin | 34.48pt | −7.52pt | −34.52pt |
| topmargin | 7.25pt | −22.75pt | −52.75pt |
| baselineskip | 12pt | 12pt | 12pt |
| parindent | 15pt | 15pt | 15pt |
| font | TeXGyreTermes (Times) | TeXGyreTermes (Times) | TeXGyreTermes (Times) |
| paper | A4 (597.5pt×845pt) | A4 | A4 |

**3p is the standard single-column Elsevier final format** — this is what Ou-Wang-Yao (2007) was published in at LAA.

### amsart vs elsarticle 3p (Ou) — Side-by-side

| | amsart 12pt | elsarticle 3p (Ou) |
|---|:---:|:---:|
| Paper | US Letter | **A4** |
| Font size | 12pt | **10pt** |
| Font | Computer Modern | **Times** |
| textwidth | 360pt (5in) | **468pt (6.5in)** |
| chars/line | ~60 | **~89** |
| lines/page | ~42 | **~52** |
| info density | 1× | **1.8×** |
| left margin | ~1.76in | ~0.9in |
| header | none by default | journal running head |

Ou's paper is visually denser and more compact because of the 30% wider text block
(468pt vs 360pt) combined with Times 10pt. The result: ~89 chars/line vs ~60,
~52 lines/page vs ~42, yielding 1.8× information density.

## Switching from amsart to elsarticle

### Compatibility

All standard LaTeX commands work across both classes:
- `\newtheorem`, `\theoremstyle` — identical
- `\DeclareMathOperator` — identical
- `\begin{proof}...\end{proof}` — identical
- `\begin{equation}`, `\[...\]` — identical
- `\input{...}` — identical

### What changes

1. **Frontmatter**: elsarticle requires `\begin{frontmatter}...\end{frontmatter}`:
   ```tex
   \begin{frontmatter}
   \title{...}
   \author[1]{...}
   \address[1]{...}
   \begin{abstract}...\end{abstract}
   \begin{keyword}
   term1 \sep term2 \sep term3
   \MSC[2020]{17B30, 17B40}
   \end{keyword}
   \end{frontmatter}
   ```
   Note: if your abstract file already wraps content in `\begin{abstract}...\end{abstract}`,
   just `\input` it directly (don't nest another abstract environment).

2. **No `\subjclass`**, no `\keywords{...}` — replaced by `\MSC[2020]{...}` and `\begin{keyword}`.

3. **Running head**: use `\title[short]{long}` instead of amsart's `\title[short]{long}`
   (syntax is identical, but elsarticle may use `\tnotetext` for title notes).

4. **`\sloppy` not needed**: the 468pt textwidth dramatically reduces overfull hboxes.
   Our 12-page amsart paper became 8 pages in elsarticle 3p with only 1 warning (1.67pt).

### Required packages for elsarticle
```tex
\documentclass[3p]{elsarticle}
\usepackage{amssymb,amsthm,amscd,mathtools}
\usepackage{newtxtext,newtxmath}     % Times fonts (bundled in tectonic)
\usepackage{microtype}
\usepackage[hyphens]{url}
```

### Switch-test results (our 834-line paper, 2026-07-11)

| | amsart 12pt | elsarticle 3p |
|---|:---:|:---:|
| Pages | 12 | **8** (−33%) |
| Hbox warnings | 3 | **1** (1.67pt) |
| PDF size | 110 KiB | **84 KiB** |
| Appearance | loose, term-paper feel | **compact, journal-published feel** |

The switch required zero changes to theorem bodies, proofs, equations, or
references. Only frontmatter was adapted.

## Resources
- AMS Author Resource Center: https://www.ams.org/publications/authors/authors
- Elsevier LaTeX instructions: https://www.elsevier.com/researcher/author/policies-and-guidelines/latex-instructions

## Visual Hierarchy in Case-Heavy Proofs (2026-07-11)

When a single proof exceeds 200 lines with 10+ subcases, it appears as a
"wall of text" — not due to content density, but because the reader's
**pre-attentive scan** finds too few anchors. Two independent axes:

1. **Hard structure** (Lemma environments, QED boxes) — per ~100 lines
2. **Soft landmarks** (bold labels, indentation) — per ~25 lines

Axes are complementary. Fixing only one leaves the other broken.

### Diagnostic method

Count hard-break anchors and soft-landmark anchors per page. A healthy
math proof has 3–4 total anchors per page. If <2, the proof needs
restructuring.

### Technique: bold subcase labels

Replace `\smallskip\noindent\textit{Subcase 1a: $u=i=1$.}` with
`\smallskip\noindent\textbf{1a.} $u=i=1$.` — italic text doesn't register
in pre-attentive scan; bold does. Zero mathematical content change.

### Technique: extract intermediate facts as lemmas

When a Claim inside a proof is referenced by multiple subcases, extract it
as a separate Lemma. Mirrors Ou's pattern (Lemma 3.1 → Theorem 3.2). The
extracted lemma's statement should be self-contained — check whether it
truly needs the outer lemma's hypotheses or can be stated more generally.
