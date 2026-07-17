# Reference Paper Structural Templates

Two published papers on derivations of strictly upper triangular matrix
Lie algebras were analyzed 2026-07-08 as structural templates. Both papers
classify linear maps (derivations, 1/2-derivations) on N_n or T_n(F).

---

## Template A: Ou, Wang, Yao (2007)
"Derivations of the Lie algebra of strictly upper triangular matrices
over a commutative ring" — Linear Algebra Appl. 424 (2007) 378–383

**Structure** (6 pages, 3 sections):
```
Title + Authors + Affiliation
Abstract + AMS + Keywords
§1 Introduction: defines N(n,R), bracket, notation M_k, α_k, β_k, N₁;
    literature review (7 citations); states aim; outlines paper
§2 Certain standard derivations: constructs 5 families (A-E) as
    building blocks — inner, diagonal, central, extremal I, extremal II
§3 Derivations: Lemma 3.1 → Theorem 3.2 (main, Step 1-6 + uniqueness)
Acknowledgments + References [1]-[7]
```

**Key patterns**:
1. **All notation in §1**. Once. Never re-introduced.
2. **Building blocks first, then assembly**. §2 constructs each
   derivation type; §3 assembles them into the main theorem.
3. **Step-based proof**. Theorem 3.2 uses "Step 1... Step 2..."
   numbering inside the proof environment. Each step has a clear
   goal: Step 1 = stabilize α₁, Step 2 = eliminate α₁, etc.
4. **No separate Motivation or Preliminaries**. Introduction serves
   both purposes. Direct, dense, 6 pages total.
5. **Concrete matrix computations**. No abstract formalism. Works
   directly with E_{ij} and bracket identities.
6. **Lemma used once, right before theorem**. Lemma 3.1 is used
   exactly once — still extracted as a lemma.

---

## Template B: Kaygorodov & Khrypchenko (2023)
"Transposed Poisson structures on the Lie algebra of upper triangular
matrices" — arXiv:2305.00727

**Structure** (12 pages):
```
arXiv ID + Title + Authors
Abstract (detailed — states all results explicitly)
Keywords + MSC2020
INTRODUCTION (unnumbered): history, literature review (~15 citations),
    states all main results, outlines paper
§1 Definitions and Preliminaries: all definitions, basic lemmas,
    general theorems (Definitions 1-3, Lemma 4, Theorem 5)
§2 Transposed Poisson structures on T_n(F):
    §2.1 Matrix algebra (basis, center, commutator)
    §2.2 1/2-derivations (Lemmas 6-9 → Proposition 10 → main results)
    Theorem 11 (n>2 case)
    Theorem 12 (n=2 case, proof omitted — references prior classification)
§3 Full matrix algebra: Proposition 13 → Theorem 14 (supplementary)
References [1]-[15]
```

**Key patterns**:
1. **Unnumbered Introduction + numbered sections**. INTRODUCTION has no
   section number; §1, §2, §3 follow.
2. **Detailed abstract**. States all main results: dim formulas,
   classification types, special cases.
3. **Preliminaries as §1**. All definitions, notations, general-purpose
   lemmas live here. Object-specific notation (matrix basis, bracket
   formulas) goes in §2.1, not in Introduction.
4. **Subsections within main body**. §2.1, §2.2 provide natural
   subdivision without creating separate top-level sections.
5. **Proposition → Theorem chain**. Lemmas build to a Proposition
   (classification of derivation space), then the Proposition feeds
   the main Theorem.
6. **Exceptional case separated**. n=2 gets its own Theorem 12; proof
   omitted because prior classification exists. Our n=4 case should
   follow this pattern.
7. **Supplementary result in own section**. Full matrix algebra result
   lives in §3, not mixed into §2. Our evaluation matrix (§6) should
   be either a separate short section or a Remark.

---

## Applying to Our Paper

### Current structure (8 sections — too many):
```
§1 Introduction (with "dim=n error" narrative)
§2 Foundations (Ψ, ᵋ, VEq, SEq, L, commutative diagram)
§3 Structure of HalfDer(N_n) (basis construction)
§4 Equation Space SEq (generators, classification)
§5 Propagation (5 subsections of main proof)
§6 Linear-Algebraic Interpretation (evaluation matrix)
§7 Main Theorem (assembly)
§8 Appendix: Lean Correspondence
```

### Target structure (3 sections + Appendix):
```
Title + Authors
Abstract (dim=n+5, basis families, 5 rigidity mechanisms)
Keywords + MSC2020: 17B40, 17B56

Introduction: define N_n, bracket, 1/2-derivation; literature review;
    state main theorem; outline paper structure
§1 Preliminaries: merge current §2-4 — basic notation, the channel
    decomposition, the four constraint families T₀-T₃, the width
    reduction and image restriction setup.
§2 A Basis of HalfDer(N_n): merge current §3 — construct n+5
    explicit 1/2-derivations organized as identity + center maps
    + boundary cocycles. Verify membership and linear independence.
§3 Proof of the Main Theorem: merge current §5+§7 into one
    section with Step 1-5 structure:
    Step 1 — Chain bridge and width reduction
    Step 2 — Diagonal propagation
    Step 3 — Image restriction (dominant reduction)
    Step 4 — Endpoint coupling and F_eq bridge
    Step 5 — Parameter count = n+5
    Then n=4 exceptional case as a separate Remark/Theorem.
Appendix: Lean-Paper Correspondence
References
```

### Key changes from reference paper patterns:
1. **Drop the dim=n error narrative**. Reference papers never discuss
   incorrect prior results in the body. If needed, a footnote.
2. **Merge the formalism**. Ψ, ᵋ, VEq, SEq are over-engineered for
   a 6-10 page paper. Reference papers work directly with bracket
   identities. Replace with concrete coefficient extraction rules.
3. **Notation once, upfront**. All channel variables (a_k, b_k, c_k),
   the ideal I, the four constraint families — define in §1.
4. **Step-based main proof**. The 5 rigidity mechanisms become
   explicit Steps inside the proof of the main theorem.
5. **Move evaluation matrix to Remark or delete**. Reference papers
   don't include redundant computational interpretations.

---

## Notation Convention Comparison

| Element | Ou-Wang-Yao | Kaygorodov-Khrypchenko | Our (current) | Our (target) |
|---------|-------------|----------------------|---------------|--------------|
| Algebra | N(n,R) (roman) | T_n(F) | N_n (mathcal) | N_n (keep) |
| Matrix units | E_{ij} (uppercase) | e_{ij} (lowercase) | E_{ij} | E_{ij} ✅ |
| Bracket | [x,y] = xy−yx | [a,b] = ab−ba | [x,y] = xy−yx | same |
| Derivation variable | φ | φ | T | φ |
| Derivation space | ad, η_x, μ_c, ρ | id, α, β_i, γ | HalfDer | Δ(N_n) |
| Basis elements | E_{ij} in α_k, β_k | e_{ij} | e_0,...,e_{n+4} | id, σ_k, β₁-β₅ |
| Identity | (part of η_y) | id | e₀ | id |
| Naming logic | By FUNCTION | By FUNCTION | By ENUMERATION | By FUNCTION |
| Formal layers | None (direct bracket) | None (direct bracket) | Ψ, ᵋ, VEq, SEq → DELETE | None |
| Ideal notation | M_n = RE_{1n} | Z([T_n,T_n]) = ⟨e_{1n}⟩ | I = span{...} | I = span{...} ✅ |

**Core lesson**: Reference papers name derivations by WHAT THEY ARE
(inner, diagonal, central, extremal type I/II) not by WHAT NUMBER THEY HAVE
in an enumeration (e₀, e₁, ...). This is the single biggest readability
difference between our current paper and the reference papers.

---

## Missing Elements Checklist

What reference papers have that ours lacks:

| Element | Ou | Kaygorodov | Our paper |
|---------|-----|-----------|-----------|
| Authors + affiliation | ✅ | ✅ | ❌ `\author{}` |
| AMS classification | ✅ | ✅ | ❌ |
| Keywords | ✅ | ✅ | ❌ |
| Literature review | ✅ (7 refs) | ✅ (~15 refs) | ❌ (0 refs) |
| Main result in Introduction | ✅ | ✅ (detailed) | ⚠️ abstract only |
| All notation in §1 | ✅ | ✅ (§1+§2.1) | ❌ scattered §2-5 |
| Step-based main proof | ✅ Step 1-6 | Lemmas→Prop→Thm | ❌ unstructured |
| Uniqueness proof | ✅ Step 6 | N/A | ⚠️ parameter count only |
| Exceptional case handling | In proof (n=3) | Separate Theorem 12 | Remark one line |
| Acknowledgements | ✅ | ✅ | ❌ |

---

## Proof Structure Comparison

**Ou-Wang-Yao** (Theorem 3.2, 6 steps):
```
Step 1: Choose r,s so α₁ is stable under φ−ρ
Step 2: Choose inner+diagonal derivations to eliminate α₁ → φ₁
Step 3: Choose inner derivation to push E_{k,k+1} into RE_{1n} → φ₂
Step 4: Choose p,q,w to eliminate E_{n-1,n} → φ₃
Step 5: φ₃ is exactly a central derivation → μ_c identified
Step 6: Uniqueness
```
Each step: (a) names the intermediate derivation (φ₁, φ₂, φ₃, ...),
(b) states the goal, (c) achieves it, (d) passes the result to the next step.
The reader always knows "we have reduced to..."

**Our current §5** (unstructured):
```
Lemma (chain bridge) → Lemma (width) → Lemma (diagonal)
→ Huge Lemma (image restriction) with 7+5 subcases
→ Subsection (endpoint reduction) → Lemma (F_eq)
→ Section §7 (main theorem assembly)
```
No φ₁, φ₂, ... naming. No "we have reduced to..." narrative.

**Target structure** (Step 1-5, following Ou):
```
Step 1: Width reduction → reduce to adjacent sources
Step 2: Image restriction → im(φ₀) ⊆ I  (dominant)
Step 3: Endpoint reduction → a_k,c_k vanish except at endpoints
Step 4: Cross-endpoint coupling → a₁+c_{n-1}=0
Step 5: Parameter tally → dim = n+5
```

---

## Voice & Style Patterns

From both reference papers:

1. **"We" voice throughout**: "we can choose", "we denote", "we obtain",
   "we see that", "it is not difficult to verify". Never first-person
   singular. Never passive-dominant.

2. **"It is easy/clear" usage**: Kaygorodov uses "it is easy to see"
   for genuinely trivial verifications. Ou uses "it is not difficult to
   verify" as a signal that the verification EXISTS but is routine.

3. **Never discuss false starts**: No "we first tried X but it failed."
   No "an earlier analysis obtained dim=n, which was incorrect." The
   paper presents only the FINAL correct argument.

4. **Dense but readable**: 6 pages for Ou, 12 for Kaygorodov. Minimal
   white space between lemmas. Transitions are one sentence, not
   paragraphs. This is the genre norm for LAA/arXiv classification
   papers.

5. **Bibliography integration**: Every claim about prior work has a
   citation. "Cao [1] studied... Doković [2] and Cao [3] described..."
   The literature review is woven into the narrative, not walled off.

---

## Writing Discipline Extracted from Reference Papers

When writing the paper, enforce:

- [ ] Every lemma/theorem preceded by a one-sentence motivation
- [ ] Derivations named by function, not by enumeration number
- [ ] "Step 1...Step 2..." structure inside the main proof
- [ ] No ∀∃∧∨ symbols in body text (write "for all", "there exists")
- [ ] No WLOG / s.t. / iff abbreviations
- [ ] No sentence starting with a math symbol
- [ ] Displayed equations have punctuation (period or comma)
- [ ] Chain equalities vertically aligned
- [ ] Three-equation syzygies displayed vertically with `\qquad` alignment
- [ ] Every parameter explicitly defined; no unspecified "constants"
- [ ] Citation includes theorem/page number: [1, Thm 3.2] or [1, §2]
- [ ] No Lean names in body text (only in Appendix)
