# Structural Wrapper Audit

Run AFTER the forward-reference audit and standard polish audits. Catches
mathematical statements that lack proper formal wrappers — a subtle class
of errors that make a paper read like "working notes" rather than a
published math paper, even when the English and formatting are clean.

## Three Checks

### Check 1: Every displayed equation with a \label must be inside a Lemma/Theorem/Proposition/Corollary environment (or its proof)

Run: `grep -n '\\label{eq:' *.tex` and verify each is inside `\begin{proof}` or
`\begin{lemma}` etc. Equations floating in free text are NOT formal results —
they're just commentary. This matters because:

- A numbered equation is an implicit claim of importance
- In a math paper, important claims must have formal wrappers
- Reviewers expect to find key results by scanning Lemma statements, not by reading commentary

**Example violation**: `c_k = 0 for 3 ≤ k ≤ n-2` with label `eq:F2interior`,
floating outside any lemma — this is a substantive result presented as informal text.

**Fix**: Wrap in a Lemma. "Endpoint reduction" becomes Lemma 3.3 with the three
equations as its statement, followed by its proof.

### Check 2: Strategy overviews must use future/reference tense, not present-fact tense

When a section begins with a proof strategy overview, any result NOT YET PROVED
must be in:
- Future tense: "We will show that..."
- Reference form: "Lemma 3.3 establishes..."
- Gerund form: "The endpoint reduction will establish..."

NOT present-fact tense: "The half-Leibniz condition forces c_k=0."

**Why this matters**: A reader who encounters "X forces Y" in present tense
reasonably expects that the proof of X→Y has already been given. If it hasn't,
the paper reads as internally inconsistent — claiming results it hasn't earned.

**Example violation**: Strategy Step 3 says "The half-Leibniz condition forces
c_k=0 for 3≤k≤n-2 and a_k=0 for 2≤k≤n-3. The b_k are free." — these results
are proved 2 pages later in the endpoint reduction. The strategy should say
"Lemma 3.3 (endpoint reduction) establishes c_k=0 for 3≤k≤n-2 and a_k=0
for 2≤k≤n-3. The b_k remain free."

### Check 3: Recurring facts need formal environments, not italic labels

When a mathematical fact is used as a premise in multiple proofs (like "no
adjacent basis element is a commutator"), it should be in a named environment:

- Best: `\begin{lemma}[Observation]` or a dedicated `observation` environment
- OK: `\begin{remark}` (if the journal has this)
- NOT OK: `\noindent\textit{Observation.}` followed by free text

**Why this matters**: Without an environment wrapper, the fact has no formal
identity. Cross-references can't cite it. The reader can't scan for it. It
blends into the surrounding commentary.

**Fix**: Define `\newtheorem{observation}[theorem]{Observation}` in the preamble,
then use `\begin{observation}...\end{observation}`. Or wrap it as a Lemma with
a descriptive name.

## Application Order

Run this after the polish checklist scans (which catch surface-level tells)
but before the forward-reference audit (which needs all wrappers in place to
trace correctly).

## Relationship to Other Audits

- **Polish checklist**: Catches surface tells (one checks, belong to, i.e.)
- **Structural wrapper audit** (this): Catches un-wrapped mathematical content
- **Forward-reference audit**: Catches symbols used before definition
- **Proof completeness audit**: Catches logical gaps in the argument chain

These form a four-layer defense: surface → structure → symbols → logic.
