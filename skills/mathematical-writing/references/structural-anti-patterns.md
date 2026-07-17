# What Doesn't Belong in a Math Paper — Structural Anti-Patterns

> Developed through iterative paper revision sessions (2026-07-11).
> These go beyond word-choice issues — they're structural patterns that
> distinguish lecture notes from research papers.

## The Core Insight

A math paper's reader is a **peer**, not a student. A lecture note's reader
is a student. Every anti-pattern below stems from confusing the two.

## Anti-Patterns Found and Fixed

### 1. Ornamental paragraphs
Facts or symmetries described but never used in proofs. Example: a
"Dynkin involution" paragraph listing symmetries of τ_k and ω_i that
advance no proof step. Fix: define concepts only where used; delete
unused facts.

### 2. Single-equation subsections
A subsection containing only one equation + one sentence of explanation.
Fix: merge into the parent subsection.

### 3. Subsection named after proof method
"Parameter tally" as a subsection title describes HOW the proof works,
not WHAT mathematical content it contains. Fix: merge the proof into
the main flow without a subsection break.

### 4. Numbered strategy overviews
"We now prove X. The proof is analyzed through the following reductions.
Step 1: ... Step 2: ... Step 3: ... Step 4: ..."
Fix: replace with a narrative paragraph stating what each lemma achieves,
not a numbered list.

### 5. Giving index roles descriptive names
"Source (i,j)" → just "(i,j)" or "E_{ij}".
"Target (u,v)" → just "(u,v)".
"Width j-i" → just "j-i" or "difference j-i".
Peers read the notation directly; students need the descriptors.

### 6. Giving equations human-readable aliases
"the chain bridge" → \eqref{eq:chainbridge}.
"the half-Leibniz condition" → \eqref{eq:halfder}.
If an equation has a number, the number IS its name.

## The Fix Pattern

For each anti-pattern, ask: "Would a peer mathematician need this to
understand the proof?" If no, delete it. The mathematics itself provides
the structure.
