# Style Audit Techniques — English Math Paper Alignment

Techniques developed during v3 alignment with Ou (2007) and KK (2023).

## 1. Explanatory Parenthetical Detection

**Problem**: Simple regex `\([^)]+\)` overcounts — it includes math notation
like `(\varphi(E_{ij}))`, `(Case~1)`, `(upper bound)`, range specs `(3\le k\le n-2)`.

**Solution**: Multi-pass filtering with prose keywords:

```python
prose_kw = [r'\busing\b', r'\bonly\b', r'\bvia\b', r'\bi\.e\.\b', r'\bsince\b',
            r'\bnote that\b', r'\brecall\b', r'\btrivially\b', r'\bapplied to\b',
            r'\bcomponent\b.*\bcancels\b', r'\bchar\b']

for m in re.findall(r'\(([^()]*)\)', text):
    stripped = m.strip()
    if len(stripped) < 6: continue
    no_math = re.sub(r'\$[^$]*\$', '', stripped)
    if any(re.search(kw, no_math) for kw in prose_kw):
        if len(re.findall(r'[a-zA-Z]{3,}', no_math)) > 0:
            # This is an explanatory parenthetical
```

**Target references**: Ou (2007) = 0, KK (2023) = 0. Our paper should also target 0.

**Pitfall**: Cross-line parentheticals can be missed. The scan above handles single-line
`(...)` only. For multiline, flatten the text first: `text.replace('\n', ' ')`.

## 2. Metalanguage Scanning

Regex patterns for tutorial/pedagogical language that Ou/KK never use:

```python
tells = [
    r'exhaust the possibilities',   # pedagogical case enumeration
    r'we split according to',       # announce-then-do pattern
    r'the following cases',         # meta-label for upcoming cases
    r'we now prove',                # "now" = conversational
    r'we distinguish',              # announce-then-do
    r'three cases',                 # counting cases for reader
    r'first.*we prove',             # sequential announcement
    r'we are now ready',            # dramatic buildup
    r'it remains to',               # checklist-style
    r'three-equation pattern',      # abstracting proof steps as "pattern"
]
```

**Target**: 0 matches in final paper.

## 3. Self-Coined Term Detection

Audit against reference paper source tex (e.g., extract KK's source via `arxiv.org/src/2305.00727`):

- Search reference source for each potential coined term
- If 0 hits in reference → it's self-coined → remove
- Replace with equation/lemma numbers or direct index notation

**Common categories**:
| Type | Example | Replacement |
|------|---------|-------------|
| Named techniques | "chain bridge" | `\eqref{eq:chainbridge}` |
| Index metaphors | "source (i,j)" | "(i,j)" or "E_{ij}" |
| Capitalized observations | "the Observation" | "Observe that..." |
| Channel metaphors | "a-channel" | "a_k-coefficient" |

## 4. Case-Label Flattening

**Problem**: Bold multi-level case labels with `\smallskip` spacing read like lecture notes:
```
\medskip\noindent\textbf{Case 1: $v<n$.}
  \smallskip\noindent\textbf{$u\ge i$.}
  \smallskip\noindent\textbf{(i)} $u=i$.
```

**Solution**: Convert all `\textbf{...}` case labels to plain `If...` paragraphs:
```
If $v<n$.
  If $u\ge i$, ...
  If $u=i$, ...
```

- Delete all `\textbf{}` wrapping on case labels
- Delete all `\smallskip`/`\medskip` before case labels
- Replace `Case 1: X` → `If X.`
- Replace `(i)`, `(ii)` → `If ...`
- Update cross-references: `by Case~1` → `by the case $v<n$`

## 5. Construction Label Alignment

If the reference paper uses (A)(B)(C) format for construction labels (e.g., Ou §2),
match it:

```latex
\textbf{Trivial derivation.}     → \textbf{(A) Trivial derivation.}
\textbf{Simple-root derivations.} → \textbf{(B) Simple-root derivations.}
\textbf{Boundary derivations.}   → \textbf{(C) Boundary derivations.}
```

The (i)-(v) sub-labels under (C) form a natural two-level nesting.

## 6. Roadmap Paragraph Removal

Ou and KK do NOT include "In Section 1 we fix... Section 2 constructs... Section 3 proves..."
roadmap paragraphs at the end of the introduction. Delete entirely.

## 7. Global Condition Compression

Instead of tutorial-style "In what follows we work under the hypotheses of Theorem 1.1.
Thus char F≠2,3 and n≥5 throughout.", compress to:

```
Throughout, $\operatorname{char}F\neq2,3$ and $n\ge5$.
```

One factual line, no meta-commentary.

## 8. Connector Density Check

Count "thus"/"hence"/"therefore"/"consequently" frequency. Target: <5 per 1000 words.
Excessive "thus" (>8/1000) reads as tutorial pacing.
