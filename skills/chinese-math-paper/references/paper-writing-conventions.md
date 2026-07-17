# Paper Writing Conventions — Learned from Ou/KK/GH/Yusupov

> Reference for mathematical paper writing style, distilled from
> comparing our paper against four published papers in the same area.
> These findings apply to English mathematical writing generally,
> not just Chinese translation.

---

## 1. Abstract Convention

**Pure results, no proof method.** All four reference papers state only
what was proved, not how:

- KK: "We describe transposed Poisson structures... We prove that..."
- GH: "we give an explicit description of all derivations of N"
- Ou: "we give an explicit description of any derivations of N(n,R)"

None say "The upper bound is obtained by restricting..." or "The lower
bound is given by constructing...". Those are proof-method descriptions
belonging in the body, not the abstract.

---

## 2. Naming Classifications Is Normal

Ou's §2 is "Certain standard derivations" with categories (A) Inner,
(B) Diagonal, (C) Central, (D) Extremal type I, (E) Extremal type II.
Giving classification names to constructed maps is standard practice.
Our "Trivial derivation", "Simple-root derivations", "Boundary
derivations" follow the same convention. Do NOT remove them.

---

## 3. Informal Language Is Widespread

Ou uses language that a strict audit would flag:
- "it is easy to check that" (3×)
- "it is not difficult to verify that" (1×)
- "get" meaning "obtain" (7×)
- "see that" (9×)
- "note that" (6×)
- "forcing" (1×)
- "As a beginning, we give a useful lemma first"

KK uses "Observe that" (6×). This is all ACCEPTED in published papers.
Do not over-correct for formality.

---

## 4. "Step" Labels Appear in Published Papers

Ou's main proof: "The proof will be given by steps. Step 1: we can
choose r,s ∈ R such that..." Explicit step numbering exists in
published work. Our removal of "Step 1/2" from Lemma 1.2 made the
paper cleaner, but the labels were not inherently wrong.

---

## 5. Proof Structure: Checklist vs Narrative

Ou's proofs flow as narrative: "Now we consider... Similar as above...
By this we may suppose..." Sentences vary in length (5-23 words).

Our original proof was a checklist: "1a. u=i=1. Then X. Thus Y.
1b. u=i>1. Then X'. Thus Y'." Uniform short sentences (6-11 words).

**When possible, prefer narrative over checklist.** Merge cases that
share the same proof template into a single flowing paragraph.

---

## 6. Case Merging — Verified by Lean

When multiple subcases use the same commuting partner and the same
bracket expansion, they can be merged into one paragraph. The
differences (how the second term vanishes) can be noted in prose
without separate subcase labels.

Verified pairing: E_{v,v+1} covers u≥i (three subcases → one).
Verified pairing: (E_{1,u}, E_{i,i+1}) covers u=i+1 and u<i,i+1<n
(two subcases → one).

Always cross-check with Lean to confirm the merge is mathematically
sound before restructuring.

---

## 7. "We" Density Is Not a Problem

| Paper | "we" count | Pages |
|-------|:--:|:--:|
| Ou | 44 | 6 |
| GH | 50 | 13 |
| KK | 10 | ~30 |
| Yusupov | 22 | ~10 |
| **Ours** | **5** | **8** |

Our "we" density is the LOWEST among all reference papers. Do not
try to reduce it further.

---

## 8. No Source/Target/Width Terminology

Zero occurrences of "source", "target", or "width" as technical terms
in any reference paper. KK uses "entry" notation a(i,j). GH uses block
notation N^{ij}. Ou just says E_{ij} and the indices directly.

---

## 9. References: 5-6 Is Normal for Short Papers

Ou (6pp): ~7 refs. Our 6 refs for 8pp is within normal range.
Zusmanovich (2010) about general δ-derivation theory is optional —
cite only if using specific results from it.

---

## 10. What NOT to Do (Checklist for Future Audits)

- [ ] No proof-method description in abstract
- [ ] No coined technique names (chain bridge, width reduction, etc.)
- [ ] No source/target/width as technical terms
- [ ] No a-channel/c-channel metaphors
- [ ] No named "Observation" paragraphs (inline "Observe that" is fine)
- [ ] No numbered Step frameworks in proof overviews
- [ ] No standalone ornamental paragraphs (Dynkin involution symmetries)
- [ ] No single-equation subsections
- [ ] No proof-method-named subsections ("Parameter tally")
- [ ] No unproven exceptional cases (n=4)
