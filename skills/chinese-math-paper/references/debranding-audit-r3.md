# Debranding Audit Round 3 — Style-Level Tells

> Added 2026-07-12 after systematic comparison with Ou (2007) and KK (2023).

Round 1 removed named techniques. Round 2 removed index metaphors.
This round targets **prose-level tells** that a blind reviewer would notice.

## 1. Explanatory Parentheticals

Ou and KK use ZERO parentheticals that explain proof steps.
If a parenthetical contains prose (not just math notation), it's a tell.

### Detection

```python
# Search for parentheticals with prose keywords
prose_kw = [r'\busing\b', r'\bonly\b', r'\bvia\b', r'\bsince\b',
            r'\btrivially\b', r'\bapplied to\b', r'\bnote that\b']
```

### Fix

Delete the parenthetical. The surrounding sentence must remain
grammatically complete and mathematically correct as a standalone claim.
If it doesn't, restructure the sentence — don't keep the parenthetical.

## 2. Metalanguage

Ou/KK never use organizational meta-language. Scan for:

| Pattern | Fix |
|---------|-----|
| "the following cases exhaust the possibilities" | Merge sub-cases into "for X... while for Y..." |
| "We now prove" | "We prove" |
| "We split according to" | Acceptable (standard math idiom) |
| "the same three-equation pattern" | "the same argument yields" |
| "In what follows we work under..." | "Throughout, ..." |
| Roadmap: "In Section 1 we... Section 2..." | Delete entirely |

## 3. Case-Label Flattening

Ou uses simple "If X, ... If Y, ..." without bold labels.
Our paper had:

```
\medskip\noindent\textbf{Case 1: $v<n$.}
\smallskip\noindent\textbf{$u\ge i$.}
\smallskip\noindent\textbf{(i)} $u=i$.
```

### Fix

Remove ALL \textbf, \smallskip, \medskip. Replace with plain "If...":

```
If $v<n$.
If $u\ge i$, ...
If $u=i$, ...
```

Any internal references to "Case 1" → "the case $v<n$".

## 4. (A)(B)(C) Alignment

Ou labels his standard derivation types as (A), (B), (C), (D).
Match this format in §2:

```
\textbf{(A) Trivial derivation.}
\textbf{(B) Simple-root derivations.}
\textbf{(C) Boundary derivations.}
```

## 5. Roadmap Deletion

Ou's introduction has NO "In Section 1 we..." roadmap.
The section titles speak for themselves. Delete any roadmap paragraph.

## Verification checklist (post-edit)

- [ ] `prose_kw` scan returns 0 explanatory parentheticals
- [ ] Metalanguage scan returns 0 (except "we split according to" which is OK)
- [ ] No bold case labels remain
- [ ] No "Case 1"/"Case 2" refs remain (use "the case $v<n$")
- [ ] §2 labels use (A)(B)(C) format
- [ ] No roadmap paragraph
- [ ] All edits compile with 0 errors
- [ ] All edits preserve mathematical correctness (cross-check with Lean)
