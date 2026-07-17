# Mathematics Paper Pedagogy for Undergraduates

Established during the `e/` project lecture notes development (2026-07-04).

## Three principles for undergraduate-oriented math writing

1. **Define everything from scratch.** Assume the reader only knows basic
   linear algebra and some Lie algebra fundamentals. Define N_n, E_{ij},
   bracket, center, ideal, Dynkin diagram, projection π_{uv} — no defaults.

2. **Discovery layer before proof.** Add 🔍 boxes explaining WHY each step
   was chosen, not just THAT it works. The reader needs to see the author's
   thought process.

3. **Concrete examples before abstract arguments.** Use n=5 throughout as
   a running example. Show bracket computations, source reductions, and
   parameter tallies for the concrete case before generalizing.

## 8 Common Student Confusion Points (and resolutions)

| # | Confusion | Resolution |
|---|-----------|------------|
| ① | Why Dynkin diagrams for nilpotent N_n? | N_n is the positive root space of sl_n |
| ② | What is a "cocycle"? | Borrowed from Lie algebra cohomology; "sparse cancelling solution" |
| ③ | How does SEq become linear algebra? | T→d² vector, SEq=row space of coefficient matrix |
| ④ | Why does image restrict to I? | "Extreme upper-right corner" elements almost commute with everything |
| ⑤ | How does bracket [E_ab, E_cd] actually work? | Domino rule: δ_bc only; first column must equal second row |
| ⑥ | What does "diagonal" mean (N_n has zero diagonal)? | Self-coefficient π_{ij}(T(E_{ij})), not matrix diagonal |
| ⑦ | Is width reduction circular? | "Breaking sticks" analogy: each split reduces width, terminates in w-1 steps |
| ⑧ | How to prove linear independence of maps? | "Test point" method: find E_{ij} and π_{uv} that isolates each e_α |

## Common mathematical errors to avoid in pedagogy

- **Self-pairing does NOT prove image restriction.** The equation
  [T(E),E] + [E,T(E)] = 0 is identically true by antisymmetry.
  Do not present this as a proof sketch.

- **dim Der(N_n) ≠ 2n-1.** Correct formula: n(n-1)/2 + 2n - 3.

- **"Diagonal" ≠ matrix diagonal.** Always clarify it means self-coefficient.

- **"Support" is jargon.** Use "non-zero action positions" for undergraduates.

## Lesson structure for maximum comprehension

```
Section Overview (半页) → Subsection Motivation (三四句) → Lemma Motivation (一句)
→ Lemma → Proof (annotated with why each step works)
→ Concrete Example (n=5) → Discovery Box → Pitfall Box
```
