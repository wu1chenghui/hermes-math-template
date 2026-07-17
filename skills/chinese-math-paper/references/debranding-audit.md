# Debranding Audit — Removing Self-Coined Technique Names

> Discovered 2026-07-11 during translation preparation for paper v3.
> The user observed that our paper coined too many technique names
> compared to the existing literature (KK 2023, Filippov 1998).

## The Problem

Math papers in this area (classification of derivations on triangular
matrix Lie algebras) do NOT give proper names to proof steps. They
reference lemmas and equations by number.

Our paper had named:
- "chain bridge" (for an equation)
- "width reduction" (for a lemma)
- "diagonal propagation" (for a lemma)
- "image restriction" (section + lemma name)
- "endpoint reduction" (section + lemma name)
- "a-channel" / "c-channel" (metaphor for coefficient directions)
- "the Observation" (capitalized, as if a named fact)

## How to Audit (Methodology)

1. **Get source tex of a key reference paper.** For KK (2305.00727):
   ```bash
   curl -sL -o /tmp/kk.tar.gz https://arxiv.org/src/2305.00727
   ```
   Extract and search for named technique patterns.

2. **Compare section titles.** KK uses purely descriptive titles:
   - "Upper triangular matrix algebra"
   - "1/2-derivations of the upper triangular matrix Lie algebra"
   NOT: "Chain bridge", "Width reduction", etc.

3. **Compare inline references.** KK says "by Lemma 2.1", "by (5)".
   We said "by the chain bridge", "by width reduction".

4. **Search for any named technique patterns.** In KK's source:
   - 0 matches for "chain", "width", "propagat", "reduction", "channel", "bridge"
   - Only "by the Jacobi" (standard named identity)

## How to Fix

| Current | Replace with |
|---------|-------------|
| Section title "The chain bridge" | "A basic bracket identity" |
| Section title "Width reduction" | "Reduction to adjacent sources" |
| Section title "Diagonal propagation" | "Equality of diagonal entries" |
| "the chain bridge" (inline) | `\eqref{eq:chainbridge}` |
| "by width reduction (Lemma X)" | "by Lemma X" |
| "a-channel / c-channel" | "$a_k$-coefficient" / "$c_k$-coefficient" |
| "the Observation" (capitalized) | "Observe that..." |
| label `eq:channeldecomp` | `eq:coeffdecomp` |

## Verification

After all changes:
- [ ] `grep` for old terms returns 0 matches in active `*.tex` files
- [ ] `tectonic` compiles with 0 errors
- [ ] Theorem/lemma/equation numbering unchanged
- Backup files (main-amsart.tex, main-els.tex) not in compilation path

## Round 2: Informal Descriptor Removal (source/target/width)

After removing formal named techniques, the user requested another pass
removing informal index metaphors: "source", "target", and "width".
These were not formally defined terms, but they create a non-KK writing
style by using metaphors where KK uses direct index arithmetic.

| Current | Replace with |
|---------|-------------|
| "source (i,j)" | "(i,j)" or "E_{ij}" |
| "target (u,v)" | "(u,v)" |
| "adjacent source" | "adjacent basis element" or "E_{i,i+1}" |
| "width j-i-1" | "difference j-i-1" or "j-i" |
| "width-2 sources" | "pairs with j-i=2" |

Evidence: KK (2305.00727) source tex contains 0 occurrences of "source",
"target", or "width" used as index descriptors. KK writes "e_{ij}" and
"the (i,j)-entry" directly. See `debranding-methodology.md` for the
three-class taxonomy (named techniques, index metaphors, capitalized observations).

### Special care needed

When removing "source" from "coefficient at a source (i,j)", the result
"coefficient at (i,j)" is unclear (coefficient functions take 4 indices).
Rewrite more explicitly: "coefficients of φ(E_{ij})" or "π_{uv}(φ(E_{ij}))".

When removing "source" and "target" from subcase labels like
"Source (1,2), target (1,v)", rewrite as "(i,i+1)=(1,2) and (u,v)=(1,v)".

### Verification

- [ ] `grep` for "source", "target", "width" (excluding label refs) returns 0
- [ ] `tectonic` compiles 0 errors
- [ ] Reading flow: index roles are obvious from surrounding math notation
