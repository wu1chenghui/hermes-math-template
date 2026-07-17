# English Mathematical Paper Writing Conventions

> Based on comparison of our paper against Kaygorodov-Khrypchenko (2023),
> Ghimire-Huang (2016), Yusupov-Madrakhimov-Vaisova (2025), and Ou-Wang-Yao (2007).

---

## Core Principle

A research paper is written for **peers**, not students. The reader does not need
help understanding structure — structure should be conveyed by the mathematics
itself. Every time you add a framing device ("Step 1", "the following reductions",
"we now turn to"), you are in lecture-note territory.

---

## What a Math Paper Should NOT Contain

### 1. Named proof techniques
Do not give formal names to individual proof steps.
- ❌ "By the chain bridge" / "By width reduction" / "By diagonal propagation"
- ✓ "By equation (5)" / "By Lemma 2.1"

### 2. Numbered step frameworks
Do not create "Step 1 / Step 2 / Step 3 / Step 4" overviews.
- ❌ "The proof proceeds in four steps: Step 1..., Step 2..."
- ✓ Let lemmas speak for themselves. The proof structure is visible from the lemma sequence.

### 3. Meta-commentary about proof structure
- ❌ "The centered part is analyzed through the following reductions"
- ✓ "Lemma 3.2 shows that im(φ₀) ⊆ I. Lemma 3.3 then constrains..."

### 4. Descriptive nicknames for mathematical objects
- ❌ "source (i,j)" / "target (u,v)" / "width j-i-1"
- ✓ Direct index notation: "(i,j)" / "(u,v)" / "j-i"

### 5. Ornamental observations
Do not include symmetries or facts that are never used in the proof.
- ❌ "The Dynkin involution exchanges ω₁↔ω₄ and ω₂↔ω₃" (when unused)
- ✓ Define σ only where it's needed (e.g., inline in ω₄ description)

### 6. Over-granular subsections
Do not create subsections for single equations or small proof steps.
- ❌ §1.2 "A basic bracket identity" (one equation)
- ✓ Merge into Notation section

### 7. Named subsections for proof method
- ❌ §3.3 "Parameter tally"
- ✓ Let the proof environment provide structure: `\begin{proof}[Proof of Theorem 1.1]`

### 8. Tutorial language markers
- ❌ "We now prove" / "let us consider" / "notice that" / "it is important to note"
- ✓ "We prove" / "Consider" / direct statements

### 9. Exceptional cases without proof
- ❌ "The case n=4 is exceptional, giving dim = 10" (if not proven in the paper or Lean)
- ✓ Only state results that are fully proven

### 10. Abstract describing proof method
- ❌ "The upper bound is obtained by restricting..." / "The lower bound is given by constructing..."
- ✓ Pure results: "dim Δ = n+5, and we construct an explicit basis"

---

## What a Math Paper SHOULD Contain

### 1. Global condition declaration
State char≠2,3 and n≥5 once, early, after the main theorem statement:
```
In what follows we work under the hypotheses of Theorem 1.1.
Thus charF≠2,3 and n≥5 throughout.
```

### 2. Lean alignment
Every lemma should have a corresponding Lean theorem. Proofs must be mathematically
identical (not necessarily structurally identical — different case decompositions are
fine as long as they cover the same logic).

### 3. Reference paper style benchmarks
| Paper | Style | Key trait |
|-------|-------|-----------|
| Kaygorodov-Khrypchenko (2023) | Lean | 0 coined terms, minimal "we", pure results abstract |
| Ghimire-Huang (2016) | Standard | 50 "we"s, section-based equation numbers, 14 proofs |
| Yusupov et al. (2025) | Standard | 22 "we"s, direct matrix classification |

---

## Audit Checklist

When reviewing a paper draft for style:

- [ ] No coined technique names used as proper nouns
- [ ] No "Step 1/2/3" framework language
- [ ] No meta-commentary ("analyzed through", "proceeds in")
- [ ] No source/target/width nicknames for indices
- [ ] No ornamental unused observations
- [ ] No single-equation subsections
- [ ] No proof-method subsection names
- [ ] No "We now prove" / "let us" / "notice that" tutorial markers
- [ ] No unproven exceptional cases
- [ ] Abstract is pure results, not proof description
- [ ] Global conditions declared once after main theorem
- [ ] All lemmas verified against Lean
- [ ] "we" count ≤ 10 (KK baseline)
- [ ] Banned words: hence, yields, vanish, belong to, one checks

---

## Key Difference: Paper vs Lecture Notes

| Lecture Notes | Research Paper |
|---------------|----------------|
| Tells reader what will happen | Shows what happens |
| Names techniques for memory | Cites lemmas by number |
| Explains motivation | States facts |
| Has overviews and summaries | Has theorems and proofs |
| "Let's try to understand..." | "We prove that..." |
| Step 1, Step 2, Step 3 | Lemma 1, Lemma 2, Lemma 3 |
