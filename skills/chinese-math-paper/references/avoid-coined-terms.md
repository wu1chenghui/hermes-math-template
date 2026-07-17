# Avoid Coined Technique Names in Mathematical Papers

> Discovered 2026-07-11 during paper v3 audit against KK (2305.00727).
> The user explicitly stated: "我们的论文不应该去过多的原创自己的术语，
> 就比如ou和kk的论文中，应该就没有出现自己创造的术语."

## The Problem

Our paper (v3, classification of 1/2-derivations on N(n,F)) gave formal
names to proof techniques:

| Coined name | What it was | KK's approach |
|-------------|-------------|---------------|
| "chain bridge" | An equation `\eqref{eq:chainbridge}` | Cited by equation number only |
| "width reduction" | Lemma 1.1 | "By Lemma 2.1" |
| "diagonal propagation" | Lemma 1.2 | No named subsection |
| "image restriction" | Lemma 3.2 | Descriptive: "the image lies in I" |
| "endpoint reduction" | Lemma 3.3 | No named subsection |
| "a/c-channel" | Coefficient metaphor | "coefficient of E_{1,n-1}" |
| "the Observation" | A basic fact | "Observe that..." |

## Evidence from KK (2305.00727)

Direct inspection of the KK LaTeX source (the closest reference paper,
also classifying 1/2-derivations on triangular matrices):

- **0 coined technique names** in the entire paper
- All references: "by Lemma 2.1", "\cref{...}", "by equation (5)"
- All section titles are purely descriptive:
  - "Upper triangular matrix algebra"
  - "1/2-derivations of the upper triangular matrix Lie algebra"
- No "Observation" with capital O
- No metaphors like "channel"

## The Fix (applied to our paper)

| Before | After |
|--------|-------|
| §1.2 "The chain bridge" | §1.2 "A basic bracket identity" |
| §1.3 "Width reduction" | §1.3 "Reduction to adjacent sources" |
| §1.4 "Diagonal propagation" | §1.4 "Equality of diagonal entries" |
| §3.1 "Image restriction" | §3.1 "Restriction to the ideal I" |
| §3.2 "Endpoint reduction" | §3.2 "Constraints on the endpoint coefficients" |
| "the chain bridge" inline | `\eqref{eq:chainbridge}` |
| "by width reduction (Lemma 1.1)" | "By Lemma 1.1" |
| "a-channel / c-channel" | "a_k-coefficient / c_k variable" |
| "the Observation" | "Observe that..." |
| `eq:channeldecomp` | `eq:coeffdecomp` |

## Principle

In a classification paper (KK/Ou/Filippov style), proof steps are
identified by lemma/equation numbers, not by coined names. Give a
technique a name only if it appears in the literature under that name,
or if it's a genuinely new concept being defined (like "centered"
derivation).
