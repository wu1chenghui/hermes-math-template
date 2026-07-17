# Comprehensive Paper Audit Framework

Six-layer, 25-check audit plan for mathematical papers. Design: pre-submission
quality gate. Execute in order — each layer builds on the previous. Blocking
issues (Layer 0) must be resolved before proceeding.

Source: executed on HalfDer(N_n) v3 paper (2026-07-09). All 25 checks passed.

---

## Layer 0: Mathematical Correctness (BLOCKING — do not proceed if any fail)

| # | Check | Method | Typical failure |
|---|-------|--------|-----------------|
| P1 | Lemma proofs correct | Algebraic verification of key equalities; test with small concrete values | Undefined notation in proof body (e.g. α_k, β_k), incorrect bracket expansions |
| P2 | Construction verifications | Hand-compute each constructed object's defining property with a small-n example | "Pointing in the right direction" without completing the verification |
| P3 | Foundational claims | Prove any assertion used as a lemma without proof (e.g. "X is not a commutator") | Unverified claims that multiple constructions depend on |
| P4 | Field characteristic usage | Audit every char≠p condition: is it actually needed? Is the only usage point correct? | char≠3 used for 3X=0→X=0 but 3X=0 appears nowhere else |
| P5 | Cross-reference integrity | Scan all \ref{} and \eqref{}, verify every label exists and points correctly | Newly-created labels (lem:centered-decomp) not referenced anywhere |
| P6 | Boundary conditions | Trace all index-range constraints at the minimal non-trivial n | "interior k: 2≤k≤n-3" is empty for n=4 |

---

## Layer 1: Forbidden Words and Negative Space (BLOCKING)

| # | Check | Method |
|---|-------|--------|
| N1 | Forbidden word scan | grep for Hence/Therefore/Consequently/Namely/i.e./vanishes/contradiction/induction/clearly/obviously/key/crucial/important/fundamental/surprising |
| N2 | Negative space 10 categories | Check each paragraph against Ou's 10 never-written categories (preview, gap narrative, self-evaluation, "Note that", restatements, examples, "as follows", inline cross-refs) |
| N3 | Deletion test | For each paragraph: delete every sentence one by one. If the argument remains complete, the sentence stays deleted |

---

## Layer 2: Sentence Structure Quantification

Target: Ou-Wang-Yao (2007) §3 benchmarks.

| # | Metric | Target |
|---|--------|--------|
| M1 | Pure math sentence ratio | ≥ 63% |
| M2 | Pure navigation sentences | ≤ 2 total |
| M3 | Grammatical subject types | ≥ 5 kinds (let/suppose, thus/so, by/applying, this/these, now, recall/note, choose, step) |
| M4 | Max subject type share | ≤ 15% (no pattern exceeds 1/7 of sentences) |
| M5 | Consecutive same subject | 0 |
| M6 | Short:medium:long ratio | ≈ 4:3:3 (short <15w, medium 15-25w, long >25w) |
| M7 | Breath after long sentence | Every >25w sentence followed by ≥1 sentence <15w |
| M8 | Breath after deep nesting | Every depth>5 sentence followed by ≥1 sentence depth 0-1 |

---

## Layer 3: Notation Audit

| # | Check | Method |
|---|-------|--------|
| S1 | Definition uniqueness | Each symbol defined exactly once (no re-definitions across sections) |
| S2 | Literature consistency | Verify notation matches reference papers (Ou/KK conventions) |
| S3 | Glyph-level consistency | Scan for forbidden variants (mathcal{N} → N_n, beta_i → omega_i, sigma_k → tau_k) |
| S4 | Equation punctuation | Every display equation ends with . or , |

---

## Layer 4: Structure and Organization

Target: Ou (2007) paper structure.

| # | Check | Target |
|---|-------|--------|
| O1 | Section count | 3 numbered sections + unnumbered Introduction. No "Conclusion" section |
| O2 | Lemma motivation | Each Lemma preceded by one sentence explaining why it exists |
| O3 | Section purpose | Each section opens with a purpose statement |
| O4 | Setup:proof ratio | ≤ 1:2 |
| O5 | Forward references | Introduction does not cite results from later sections |
| O6 | Lower+upper bound closure | §2 constructions are cited in §3's closing argument |
| O7 | Small-n handling | One-sentence dismissal; no separate proof for trivial cases |

---

## Layer 5: LaTeX and Typesetting (COSMETIC — non-blocking)

| # | Check |
|---|-------|
| L1 | tectonic compilation: 0 errors |
| L2 | No stray punctuation (`,.`, `,,`) |
| L3 | Reference format matches target journal convention |
| L4 | Single theorem counter (Theorem 2.1, Lemma 2.2, not Theorem 2.1 + Lemma 2.1) |

---

## Execution Order

1. **A-group scan** — grep all files for forbidden words, display eq punctuation,
   undefined refs. See `references/per-file-audit-template.md` for patterns.
2. **B/C-group read** — per-file, per-sentence, per-paragraph. The template in
   `references/per-file-audit-template.md` provides the exact questions to ask
   at each level.
3. **Layer 0 P1-P6** — blocking: mathematical correctness (Lean verification,
   foundational claims, char usage, refs, boundaries)
4. **Layer 1 N1-N3** — blocking: voice and style
5. **Layer 3 S1-S4** — notation audit
6. **Layer 2 M1-M8** — sentence metrics
7. **Layer 4 O1-O7** — structural checklist
8. **Layer 5 L1-L4** — final compilation
9. **A-group rescan** — re-run grep after all fixes. Fixes can introduce new
   forbidden words (e.g., "i.e." discovered only on second pass).

---

## Representative Anti-Patterns (from executed audit)

These are the most common failures discovered during cold-read analysis:

1. **Meta-commentary**: "This identity is the central computational tool of the paper." → Just present the identity.
2. **"Organized as follows" paragraphs**: PhD thesis trope, never in published papers.
3. **Triple definitions**: Same object defined in intro, §1, and §3. Define once.
4. **Sketched verifications**: "Commuting pairs contribute zero by (I)" without showing the cancellation mechanism.
5. **Forward references outside section**: "Section 3 proves im(φ)⊆I" in the introduction.
6. **Imprecise verbs**: "yields", "survives", "dispatches" → "gives", "is non-zero", "we consider cases".
7. **Wrong cross-references**: "E_{1,n} is central by (I)" — centrality is a Lie algebra property, not a consequence of how I is defined.
8. **Abstract tells**: "five local rigidity mechanisms" → "five additional constraints"; "formalized in Lean" → move to acknowledgments.

## New Pitfalls (2026-07-11 session, full audit pass)

9. **Bounds mismatch between strategy overview and body**: The strategy
   overview said `c_k=0 for k≥3` but the endpoint proof says `3≤k≤n-2`.
   The summary omitted the upper bound, implying `c_{n-1}=0` which contradicts
   the surviving variable list. **Rule**: strategy overviews must replicate
   the exact index ranges from the proof they summarize.

10. **"Maximal"/"Minimal" claims without verification**: "This ideal is
    maximal abelian" was stated but false — `I + ⟨E_{1,n-2}⟩` is also
    abelian. **Rule**: any claim containing "maximal" or "minimal" must be
    explicitly verified — these terms are rarely casual in classification
    papers and are often false when claimed offhand.

## Gap-to-Appendix Pipeline

When the audit finds completeness gaps (sketched verifications, missing bracket
expansions, "a 2×2 system forces..." without showing the system):

1. **Fill the gap in the body first** — make the proof self-contained. Show ONE
   representative bracket expansion in full detail so the reader sees the pattern.
2. **Move verbose computations to the appendix** — if filling every subcase would
   bloat the body (e.g., 5 subcases each requiring 20 lines of bracket algebra),
   show the representative case in the body and put the remaining expansions in
   an appendix under "Supplementary computations."
3. **Structure the appendix by subcase** — mirror the body's case labeling
   (1a, 1b, 1c, ...) so the reader can cross-reference effortlessly.
4. **Include the chain-bridge recurrence** — for bracket-manipulation proofs,
   the chain bridge (or equivalent recurring identity) should be expanded once
   in the appendix with all index cases shown.
5. **Verify Dynkin/automorphism invariances** — if the proof uses symmetry under
   an involution, the appendix should contain the verification that the
   involution preserves the relevant structure (e.g., σ preserves
   1/2-derivations).

The target: a referee who wants to check every detail can find it in the
appendix; a reader who trusts the pattern can skip the appendix and follow
the body's representative computation.
