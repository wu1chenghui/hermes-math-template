# Sources for Mathematical Writing Principles

This file archives the original research gathered during the 2026-07-08
session. The synthesized principles live in `../SKILL.md`.

---

## Source 1: Terry Tao — "On Writing" (wordpress.com)

Four articles from https://terrytao.wordpress.com/advice-on-writing-papers/

### 1a. Organise the Paper

Key excerpts:
- "A stream-of-consciousness format, in which results are presented in the
  order in which they occurred to the author, are generally a very bad idea"
- "Whenever possible, each major milestone in the argument should be
  formalized in a self-contained and prominently located proposition or
  theorem, in order to facilitate a 'high-level' understanding"
- "Readability is improved if these milestones are placed relatively early in
  the paper (and, in particular, before all the technical details involved in
  the proof of that milestone have appeared yet)"
- "Papers in which the punch line is delayed until the very last page of the
  argument tend to be particularly frustrating to read"
- "The more technical components of the paper should be pushed to the back of
  the paper if possible, in order to ease the 'learning curve'"
- "For large papers: draw a dependency DAG on a blackboard — boxes for each
  lemma/theorem, arrows for each logical deduction"

### 1b. Motivate the Paper

Key excerpts:
- "A paper should not just be a sequence of formulae or logical steps. It
  should also be organized and motivated in such a way that the reader is
  always aware what the near-term and long-term objectives are"
- "Informal, heuristic, or motivational reasoning is therefore very welcome,
  but should be clearly indicated as such"
- "Before presenting your most general result, it can help to first discuss a
  less technical special case or 'toy' result first"
- "At the start of each section, it is often a good idea to give a brief
  paragraph describing the purpose of that section"

### 1c. Take Advantage of the English Language

Key excerpts:
- "Just because you can write statements in purely mathematical notation
  doesn't mean that you necessarily should"
- "English sentences can be considerably more expressive [than mathematical
  expressions]"
- Demonstrated 20+ ways to express "P(x) ∧ Q(y)" in English, each carrying
  different emphasis, causality, importance, and context
- Key signal words: Therefore (consequence), Moreover/Furthermore (addition),
  In particular (special case), However (contrast), Unfortunately/Fortunately
  (valence), For future reference (bookmark), Meanwhile (parallel),
  Equivalently (equivalence), More generally (generalization)

### 1d. Write in Your Own Voice

Key excerpts:
- "One should NOT go so far as to copy entire paragraphs or more of text
  from a prior paper"
- "Paraphrase and interpret the previous text rather than copy verbatim"
- "Maintain some level of notational consistency with the previous literature"
- "If an argument is standard in the literature, structure it in a standard
  fashion if possible, again to assist the expert readers"

---

## Source 2: Paul Halmos — "How to Write Mathematics" (1970, AMS)

Core philosophy: "The basic problem is to communicate an idea. To do so, and
to do it clearly, you must have something to say, you must have someone to
say it to, you must organize what you want to say, you must arrange it in the
order you want it said, you must write, rewrite, and rewrite it again several
times, and you must be willing to work hard on mechanical details."

Ten rules:
1. Say something — have content
2. Speak to someone — know your audience
3. Organize — minimize reader resistance, maximize insight
4. Use consistent notation — notation deserves careful thought
5. Write in spirals — constantly revise earlier sections
6. Watch your language — grammar, word choice, punctuation
7. Be honest — anticipate difficulties, defuse in advance
8. Remove the irrelevant — "Don't dress up trivialities as theorems"
9. Use words correctly — think about logical "small words"
10. Resist symbols — "The best notation is no notation"

Additional: introduced the tombstone symbol (∎, "halmos") to mark end of proofs.

---

## Source 3: AMS Style Guide

Based on Ellen Swanson's *Mathematics into Type*.
Available at: https://www.ams.org/arc/styleguide/index.html

Three parts:
- Part 1: Structure (article format)
- Part 2: Editing and Style (text and math editing)
- Part 3: Appendices (function lists, mathematician names, abbreviations)

Key conventions:
- Theorem environments: Theorem, Lemma, Corollary, Proposition, Definition,
  Example, Remark, Note, Conjecture, Problem, Question
- Numbering: by section (`\numberwithin{theorem}{section}`) or sequential
- Citation systems: numeric (most common), author-year, alphanumeric
- MR numbers mandatory in references
- Font conventions: functions in roman, variables in italic, number sets in
  blackboard bold, vectors in bold
- Document classes: amsart, amsproc, amsbook

---

## Source 4: Keith Conrad — "Advice on Mathematical Writing"

Located at: https://kconrad.math.uconn.edu/blurbs/proofs/writingtips.pdf
(10-page PDF — fully extracted 2026-07-08 via pypdf, 22,834 chars)

Nine sections with concrete Bad/Good examples:
1. Notation (11 rules): don't start sentences with symbols, separate symbols
   with words, no ∀∃∧∨, no WLOG/s.t./iff, one meaning per variable, define
   all notation, notation fits context (p for primes, x for set elements)
2. Equations: differentiate equation vs expression, display important equations
   with labels, chain equalities vertically, punctuate equations as sentences
3. Parentheses and commas: no irrelevant parentheses, "Let ..., then ..." is
   always wrong
4. Helpful words: tell reader where you're going, use key words (since,
   because, on the other hand), vary word choice
5. English grammar: it's/its, sentence fragments, verb tenses (math facts are
   present tense), singular/plural, spelling (counterexample is one word)
6. Types of results: Theorem > Lemma > Corollary ordering, capitalize when
   numbered ("Theorem 2.1")
7. Definitions: term only, no properties or examples inside definition env
8. Fonts: single-letter variables italic, numbers never italic, multi-letter
   functions roman, theorem statements italic
9. LaTeX tips: URL escaping, left double quotes, multiplication symbols

## Source 5: Bjorn Poonen — "Practical Suggestions for Mathematical Writing"

Located at: https://math.mit.edu/~poonen/papers/writing.pdf
(6-page PDF — fully extracted 2026-07-08 via pypdf, 13,964 chars)

65 numbered rules organized into: (1) Important things — explain what each
claim follows from, eliminate ambiguity, break arguments into lemmas, make
quantifiers unambiguous; (2) Title/abstract/intro hierarchy — each describes
the whole paper with increasing detail, omit "A note on", no conclusions
section; (3) Style — "clear" is usually wrong, no WLOG/iff/s.t., refer by
number not "above", short sentences; (4) LaTeX — single numbering system,
\\DeclareMathOperator, \\colon; (5) Nitpicks — "so that" vs "such that",
semicolon rules, principal vs principle, spelling

## Source 6: Jean-Pierre Serre — "How to Write Mathematics Badly"

Transcript URL: https://bpb-us-w2.wpmucdn.com/web.sas.upenn.edu/dist/0/713/files/2025/05/How_to_write_mathematics_badly__transcript_.pdf
Video: https://www.youtube.com/watch?v=ECQyFzzBHlo
(16-page transcript — fully extracted 2026-07-08 via pypdf, 34,606 chars)

Ironic lecture (Harvard 2003). Key warnings: (1) "a"→"the" switch: Theorem 1
says "there exists a map" but Theorem 2 refers to "the map" as if uniquely
chosen — construct objects first, then state properties; (2) Commas doing
double duty as "and" and "hence"; (3) Symbols replacing verbs (∈ as the only
verb in a theorem); (4) Unspecified constants ("C a constant" — depends on
what?); (5) Diagrams claimed commutative without proof; (6) The empty proof
symbol □; (7) "A proof is accepted by experts; a Bourbaki proof is accepted
by non-experts"; (8) Identifications (N⊂Z⊂Q⊂R) are lies we accept; (9) Prefer
detailed proofs over idea sketches

## Source 8: Lean-verified Endpoint Reduction (F1/F2 Pattern)

See `references/F1F2-endpoint-pattern.md` for the Lean-verified structure
of the endpoint reduction in derivation-classification papers.  Key insight:
constraints come from two commuting-pair half-Leibniz equations (F1 and F2),
not from chain-bridge recurrences.  Boundary cases require careful handling
of index overlap with the ideal I.

Located at: https://www.math.ucla.edu/~pak/papers/how-to-write1.pdf
(18-page PDF — fully extracted 2026-07-08 via pypdf, 58,700 chars)

Modern perspective: (1) Clarity as competitive advantage in arXiv era, for
non-native readers with short attention spans; (2) Final Remarks as endnotes
— cite most related work here, not in body; (3) 11-level citation hierarchy
from definitive to personal communication; (4) Alphanumeric references
[SY09], no BibTeX; (5) Downshift style — ten "therefore"s fine, short
sentences, present tense, standard American spelling; (6) LaTeX macros for
everything; (7) Figures: many but small (3-5 cm); (8) Let paper stew a week,
update arXiv after publication, advertise via blog/talks

---

## URLs for Future Research

- Steven Krantz: *A Primer of Mathematical Writing* and *Handbook of Writing
  for the Mathematical Sciences*
- Donald Knuth: *Mathematical Writing* (Stanford CS209 course notes)
- Nicholas Higham: *Handbook of Writing for the Mathematical Sciences*
- Dimitri Bertsekas: "Ten Simple Rules for Mathematical Writing"
- MAA (Mathematical Association of America) writing guidelines
