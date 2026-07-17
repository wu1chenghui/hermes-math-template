# Explanatory Parenthetical Audit

## What they are

Parenthetical phrases in a math paper that explain *how* or *why* a result
follows, rather than just parenthesizing math notation.  Examples:

- `(using d₃ = d₁, d₄ = d₂ from above)` — reminding reader of substitution
- `(only the bracket-right term contributes, via [E₁ᵤ, Eᵤₙ] = E₁ₙ)` — partial proof inside parens
- `(only c_k E₂ₙ interacts with E₁₂)` — qualifying which term matters
- `(via [E₁,ₙ₋₁, Eₙ₋₁,ₙ] = E₁ₙ)` — explaining mechanism

## Why they matter

Ou (2007, 6pp) and KK (2023, 12pp) use **zero** explanatory parentheticals.
When our paper has 4 across 5 pages, it's a stylistic deviation from the
reference papers.  The reader of this genre is expected to verify routine
computations themselves.

## Detection methodology

### Step 1: Regex extraction (LaTeX source)

```python
import re

prose_keywords = [
    r'\busing\b', r'\bonly\b', r'\bvia\b', r'\bi\.e\.\b',
    r'\bsince\b', r'\bnote that\b', r'\brecall\b', r'\btrivially\b',
]

matches = re.findall(r'\(([^()]*)\)', text)
for m in matches:
    m = m.strip()
    if len(m) < 6:
        continue  # skip short math-only parens like "(Eij)"
    has_prose = any(re.search(kw, m) for kw in prose_keywords)
    # Remove math blocks and LaTeX commands; check for remaining English words
    no_math = re.sub(r'\$[^$]*\$', '', m)
    no_math = re.sub(r'\\[a-zA-Z]+(\{[^}]*\})*', '', no_math).strip()
    if has_prose and len(re.findall(r'[a-zA-Z]{3,}', no_math)) > 0:
        # This is an explanatory parenthetical
```

### Step 2: Compare against reference papers (PDF via pymupdf)

```python
import fitz
doc = fitz.open("reference.pdf")
for page_num in range(doc.page_count):
    text = doc[page_num].get_text().replace('\n', ' ')
    # Apply same regex as Step 1
```

### Step 3: Manual verification

The keyword filter is a heuristic — review each match to confirm it's
genuinely explanatory (not just a math expression that happens to contain
a keyword).

## Reference paper comparison (2026-07-12)

| Paper | Explanatory parens | Pages |
|-------|:---:|:---:|
| Our paper (v3, pre-fix) | 4 | 5 |
| Our paper (v3, post-fix) | 0 | 5 |
| Ou-Wang-Yao (2007) | 0 | 6 |
| Kaygorodov–Khrypchenko (2023) | 0 | 12 |

## Decision framework

When you find an explanatory parenthetical:

1. **Can the sentence stand without it?**  If `gives πᵤₙ = 0` is already a
   complete claim, the parenthetical is pure scaffolding. → **Delete.**
2. **Is the bracketed content a genuine computation step?**  If so, ask
   whether the target reader can fill it in themselves.  For a Lie-algebra
   classification paper, the answer is almost always yes. → **Delete.**
3. **Would "absorbing into the sentence" help?**  No.  A sentence like
   "contributes a_k via [E₁,ₙ₋₁, Eₙ₋₁,ₙ] = E₁ₙ" is STILL an explanatory
   parenthetical — you just removed the brackets.  The computation detail
   is the problem, not the punctuation. → **Delete the whole clause.**
4. **Is the "using / via" clause load-bearing for the argument?**  If a
   reader genuinely cannot follow without it, the surrounding exposition
   needs to be rewritten, not patched with a paren. → **Rewrite the
   paragraph, don't add a paren.**

## Pitfall: overcounting

A naive regex `\([^)]{8,}\)` will count ALL parentheticals, including
`(E_{ij})`, `(Case~1)`, `(3 ≤ k ≤ n-2)`, etc.  These are math notation,
not explanation.  Always filter with the prose-keyword heuristic and
manual verification.

## Pitfall: "absorbing" looks like a fix but isn't

In the 2026-07-12 audit, parenthetical #4 was initially classified as
"absorb into sentence":
```
contributes a_k at (1,n) (via [E₁,ₙ₋₁, Eₙ₋₁,ₙ] = E₁ₙ).
→ contributes a_k at (1,n) via [E₁,ₙ₋₁, Eₙ₋₁,ₙ] = E₁ₙ.
```
On re-examination, the preceding sentence already says "a_k-coefficient
analogue of the previous computation" — the reader already knows what
computation to perform.  The `via` clause is redundant scaffolding.
**Correct fix: delete entirely.**
