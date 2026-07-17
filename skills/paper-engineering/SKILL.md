---
name: paper-engineering
description: >-
  Turn a Lean-formalized proof into a journal-ready mathematical paper.
  NOT translation. Pipeline: Proof Cards → Math Story → Paper Draft →
  Compressed Paper → Final Paper. Covers proof reconstruction, audit
  methodology, and paper writing discipline.
---

# Paper Engineering — From Lean Formalization to Journal Paper

## When to load

- User asks to "write a paper" from a Lean formalization
- User wants to audit a paper draft for logical vulnerabilities or Lean consistency
- User asks for proof reconstruction, compression, or narrative design
- User mentions "paper engineering", "proof cards", "math story"

## Core Principle: Lean is a Proof Oracle, NOT a Draft

NEVER translate Lean → paper directly. The paper must be a self-contained
mathematical argument. Lean certifies correctness; the paper provides
the human-readable proof.

```
Lean Code (read-only proof oracle)
    ↓
Proof Cards (each theorem: purpose, role, dependencies)
    ↓
Math Story (why the proof works, no Lean, no code)
    ↓
Paper Draft (mathematical language, unified notation)
    ↓
Compressed Paper (many Lean theorems → few math statements)
    ↓
Final Paper (journal language, Appendix = Lean verification index)
```

## Phase 1: Proof Cards

For every Lean theorem in the active proof chain:

```
Name: <Lean theorem name>
Purpose: WHY this theorem exists. One sentence about its role.
Depends on: <theorems used>
Used by: <theorems depending on it>
Paper section: <where it belongs>
```

Rule: Purpose, not Statement. Focus on the conceptual pillars. For the
1/2-derivation classification: Reduction / Confluence / Propagation.

## Phase 2: Math Story

Write a narrative answering "Why does the entire proof work?" — zero
Lean code, zero theorem names. A mathematician who has never seen Lean
should understand the proof structure. Key question for every section:
"Why does the reader need this before the next section?"

## Phase 3: Paper Draft

Critical rules:
- Section titles: mathematical concepts, NOT Lean module names ("The
  Reduction Scheme" not "ReductionTree")
- Notation: unified throughout, one system, no symbol drift
- Language: mathematical prose, not development prose
- Lean references: ONLY in Appendix; at most marginal notes in body
- Proofs: mathematical paragraphs, never tactic sequences

## Phase 4: Compression

Compress many Lean theorems into few mathematical statements. Many Lean
theorems are separate because the proof assistant needs atomic steps;
a mathematician sees one observation. Target: 40+ Lean theorems → 15-20
paper statements. Do NOT compress genuinely distinct mathematical ideas.

---

## Audit Framework

### Six-Layer Paper–Lean Verification

Run in order. Each layer answers a different question:

```
Layer 0 — Definition Audit
  Q: Do Lean definitions match mathematical definitions?
  Check: bracket formula, HalfDerivation condition, Phi, coeffOf, etc.

Layer 1 — Lean Internal Soundness
  Q: Is the Lean proof itself mathematically sound?
  Check: statement accuracy, char propagation, hidden assumptions

Layer 2 — Completeness Audit (Dominator Classification)
  Q: Are all Lean dominator nodes present in the paper?
  Classify every Lean theorem as A/B/C:
    A: Dominator — main theorem, skeleton nodes → FULL proof required
    B: Supporting — local scope → compressed but present key mechanism
    C: Technical — automation, bookkeeping → skip in paper
  Output: P0/P1/P2 gap list + numbered fix roadmap

Layer 3 — Proof Reconstruction Audit (Node-level)
  Q: Does each node use the SAME proof mechanism?
  Check: same HL equations? same variables? same resolution?

Layer 4 — Proof Trace Audit (Edge-level)
  Q: Does every dependency edge have a bridge? Any new reasoning?

Layer 5 — Proof Replay (Line-level)
  Q: Does every paper sentence map to specific Lean proof lines?
  Rule: every Lean `have` that is not pure bookkeeping needs a
  corresponding sentence in the paper.
```

Execute Layer 0-1 first (verify Lean), then Layer 2-5 (verify Paper↔Lean).
Layers 0-3 are mandatory before editing the paper. Layer 4 before submission.
Layer 5 is the gold standard for formal-verification-level verifiability.

### A/B/C Consistency Audit (Quick pre-edit check)

Evaluate every theorem on three independent dimensions:
- **A. Correctness**: Is the proof VALID? (tautologies as constraints? circularity?)
- **B. Completeness**: Is the proof FULLY WRITTEN? (case dispatch? bracket expansions? "此处从略"?)
- **C. Lean Consistency**: Does the paper's argument match what Lean proves?

Priority: 🔴 P0 (wrong/missing) → 🔴 P1 (Lean mismatch) → 🟡 P2 (compressed) → 🟢 P3 (formatting)

### Referee Attack Audit

Seven rounds, performed as HOSTILE REFEREE:
1. Hidden assumptions ("thus", "hence", "clearly", "by induction", "wlog")
2. Quantifiers (∀ in statement → ∀ in proof)
3. Boundary cases (n≥5, width=1/2/3/4, edge indices)
4. Induction legitimacy (recursive calls strictly decrease measure)
5. Cancellation (field ops valid in char≠2; division-by-2 check)
6. Dependency minimality (each theorem uses only what it claims)
7. Full hostile pass (read every sentence trying to reject)

---

## Proof Reconstruction from Lean

When a paper proof is structurally broken (not just compressed), use 4-stage
workflow. Do NOT start editing the paper — rebuild the proof first.

### Stage 1: Reconstruction Plan
Extract mathematical statements of every Lean helper lemma. Verify the proof
tree is mathematically minimal. Document: each helper's statement, HL
equations, char-condition, closing mechanism.

### Stage 2: Rebuild Each Helper (leaf-first)
Prove each helper as an independent mathematical argument. Rules:
- Derive from half-Leibniz axiom + bracket formula, not from Lean code
- Every "only this term survives" claim → Type I/II analysis
- Ad-hoc case enumeration ("if p=1… if q=i…") is FORBIDDEN
- After each helper: 4-question Lean audit

### Stage 3: Assemble
Wire helpers following the DAG. Main proof becomes "by case dispatch."

### Stage 4: Paper Rewrite
Only NOW edit the paper. Proof is established; rewrite is organization.

### Lean Audit (per helper, 4 questions)
1. Same number of HL equations used?
2. Same source/partner/target triples?
3. Same linear relations derived?
4. Any extra hypotheses (centeredness, I_filtered, width induction)?

### Unified Helper Proof Template
Every helper lemma follows this fixed template:
1. Statement (mathematical proposition, hypotheses, conclusion)
2. Choice of HL equations (explicit src/ptr/tgt triples)
3. Coefficient extraction via bracket formula (BracketSource/Left/Right)
4. Linear relations
5. Elimination
6. Lean audit (4 questions)

---

## Type I/II Bracket Analysis Framework

For bracket formula [E_{p,q},E_{r,s}] = δ_{q,r}E_{p,s} − δ_{s,p}E_{r,q}
at fixed target (u,v), exactly two mutually exclusive types:

- **Type I**: δ_{q,r}=1 AND (p,s)=(u,v) → contribution +1·coeff
- **Type II**: δ_{s,p}=1 AND (r,q)=(u,v) → contribution −1·coeff

All other cases contribute zero. State this once as an Observation at the
start of §5, then invoke "by the Type I/II criterion" in every subsequent
bracket analysis. No case enumeration needed.

### Pattern Extraction

Extract proof patterns BEFORE writing individual lemmas. Classify helpers
into 3-5 patterns (3-equation linear system, single-equation elimination,
2-equation syzygy, width induction, reduction to prior pattern). If two
helpers differ only in index substitution, they belong to the same pattern.

---

## Writing Discipline

### Section Responsibility

Each section does exactly one thing:
- **§2**: Define objects only. No proofs.
- **§3**: Lower bound only. Not the upper bound mechanism.
- **§4**: Introduce constraint space concepts. State results proved later.
- **§5**: ALL constraint reduction proofs. Each subsection = one reduction
  layer. Never counts dimensions, never names basis vectors (B,C,D,E,F),
  never states the main theorem.
- **§6**: Interpretation only. No proof obligation. No new mechanisms.
- **§7**: Assembly only. Invokes results from §3 and §5. No new proofs.
  Must include explicit parameter tally.
- **§8**: Appendix only. Lean correspondence.

### Freeze Architecture Before Rewriting

When proof sketch has incorrect mechanism claims:
1. Create Proof Architecture doc (every layer, rank contribution, Lean
   correspondence, logical order)
2. Review for cycles and unique responsibilities
3. Freeze the architecture
4. THEN rewrite sections following the frozen architecture

---

## Patch Design

When fixing completeness gaps identified by audit, do NOT immediately
start writing. First produce a Patch Design:

- **Location**: exact section and insertion point
- **Size**: estimated line count
- **Purpose**: which audit gap it closes
- **Risk**: narrative disruption, forward references, dependency issues
- **Dependencies**: what other sections/lemmas it references

Execution order: resolve cross-patch dependencies first, then highest-
priority patch (induction closure, final assembly). Stop when the paper
is the shortest complete proof — not when every design-archive lemma
is transcribed.

---

## Pitfalls

### GitHub README math rendering
When publishing the paper's README on GitHub, four rendering traps break
LaTeX: `\operatorname` unsupported, `$` in headings, `_` in table cells,
and CJK punctuation touching `$`. Full reproduction recipes in
`references/github-readme-math-rendering.md`.

### Self-pair identity
[T(E),E] + [E,T(E)] = 0 expands via antisymmetry to 0=0. This is a
TAUTOLOGY — provides ZERO constraint. Any proof invoking self-pair to
"force coefficients to zero" is WRONG (P0 correctness error). In the
endpoint reduction, once the image is in $I$, self-pair is automatically
satisfied because $[I,E_{k,k+1}]=0$ for interior $k$ and boundary
cancellations follow from the explicit coefficients. No separate
self-pair paragraph is needed in the paper.

### Lean delegation
"验证在 Lean 形式化中已完成，此处从略" is FORBIDDEN. The paper must
contain a complete mathematical proof. Lean is verification, not
substitute. Every claim must be backed by explicit computation or a
self-contained argument — never by "see Lean formalization."

### Boundary off-by-one in channel constraints
When deriving constraints like $c_k=0$ for commuting-pair equations,
the boundary cases ($k=n-1$ for the right endpoint, $k=1$ for the left)
often give a DIFFERENT equation (typically the cross-endpoint coupling
$a_1+c_{n-1}=0$) rather than forcing the variable to zero. Always check
whether the claimed range actually includes the boundary indices.

Pattern: "for $k\ge3$" → verify $k=n-1$ separately; "for $k\le n-3$"
→ verify $k=1$ separately. This caused a genuine mathematical error
in our paper where $c_{n-1}$ was incorrectly claimed to be zero.

### Non-commuting pair disguised as $\Psi=0$
$\Psi(x,y,\varphi)=[\varphi x,y]+[x,\varphi y]=0$ is the half-Leibniz
for COMMUTING pairs only. For non-commuting pairs, the full equation is
$2\varphi[x,y]=\Psi(x,y,\varphi)$. Writing $\Psi=0$ for a non-commuting
pair is a mathematical error. Always verify $[x,y]=0$ before using
$\Psi=0$. Failure case: $[E_{i+1,u},E_{i,i+1}]=-E_{i,u}\neq0$ when
$u>i+1$ (Case 2(v) of image restriction).

### F1/F2 constraint structure (endpoint reduction)
The channel constraints in the endpoint reduction come from two
half-Leibniz equations applied to each adjacent source, NOT from a
chain-bridge recurrence:

- **F2**: pair $(E_{k,k+1},E_{12})$ at target $(1,n)$ → $c_k=0$ for
  $3\le k\le n-2$, and $a_1+c_{n-1}=0$ at $k=n-1$
- **F1**: pair $(E_{k,k+1},E_{n-1,n})$ at target $(1,n)$ → $a_k=0$ for
  $2\le k\le n-3$, and $a_1+c_{n-1}=0$ at $k=1$

The chain bridge at adjacent pairs $(E_{k,k+1},E_{k+1,k+2})$ gives
ZERO at targets $(1,n)$, $(1,n-1)$, $(2,n)$ — it does NOT couple
adjacent channel variables. Any claim of a chain-bridge recurrence
$2c_{k,k+2}=c_k+c_{k+1}$ is incorrect; both bracket terms vanish
at these targets for interior $k$.

The $b_k$ coefficients (coefficients of $E_{1,n}$) are structurally
absent from both F1 and F2 because $E_{1,n}$ is central — $[E_{1,n},\cdot] =
[\cdot, E_{1,n}] = 0$. The Lean Spanning.lean confirms: `coeff k (k+1) 1 n`
does not appear in `F1_coeff_relation` or `F2_coeff_relation`.
Paper prose: "The $b_k$ are not constrained by either commuting pair,
because $E_{1,n}$ is central and its brackets with $E_{12}$ and
$E_{n-1,n}$ are zero."

### Reconstruct vs. Complete
- **Complete**: direction correct, steps omitted → fill in steps
- **Reconstruct**: mechanism WRONG or NONEXISTENT → start from scratch
Do NOT conflate these. Cosmetic fixes on a broken proof waste time.

### Appendix correspondence table errors
Common: mapping lemma to wrong Lean theorem, omitting critical lemma,
listing theorem that proves something different. Always `lean_verify`
before trusting the table.

### Nullspace verification
BEFORE writing the paper: set up full constraint matrix over Q for small n,
Gaussian-eliminate → nullspace dimension. If ≠ paper claim, STOP. The N_n
project claimed dim=n; nullspace revealed dim=n+5.

### "Only this term survives" without proof
Every such claim must be backed by Type I/II analysis showing which type
survives and which is blocked by an explicit index inequality.

---

## When to Stop Auditing

After the Blind Referee Audit passes and Paper↔Lean consistency is
verified, STOP inventing new audit types. Remaining work is:
- English language polishing
- Notation conflict resolution
- Overfull hbox fixes (LaTeX cosmetic)
- Target journal formatting
- Final PDF read-through

### Post-Freeze Editing Criterion

Only modify when two or more independent readers report the same
misunderstanding at the same location. Most reviewer feedback at this
stage is preference, not correctness.

---

## Writing Like a Mathematician

When the paper reads like a Lean transcript or a development log rather than
a mathematical paper, **load the `mathematical-writing` skill** — it carries
all writing principles and a pre-submission checklist. This skill covers
paper organization, motivation, English usage, notation, voice, AMS
formatting, and modern 21st-century practices. Synthesized from Tao
(4 articles), Halmos (10 rules), AMS Style Guide, Conrad (notation rules),
Poonen (65 practical rules), Serre (ironic lecture transcript), and Pak
(21st-century tips). For post-draft sentence-level auditing, use
`mathematical-writing` skill references: `sentence-structure-metrics.md`
(8 quantifiable targets) and `negative-space-checklist.md` (10 deletions).

For concrete structural templates from published papers in the same genre
(derivation classification on matrix Lie algebras), see
`references/reference-paper-templates.md`. It analyzes Ou-Wang-Yao (2007)
and Kaygorodov-Khrypchenko (2023) as models for section structure, notation
conventions, main-proof organization, and voice.

For sentence-level writing DNA (forbidden words, bracket calculation templates,
two computational philosophies, deduction verb spectra, forensic proof
reconstruction), load the `mathematical-writing` skill and read
`references/reference-paper-analysis.md`.

---

## Reference Files

- `references/audit-checklist.md` — 4-dimension rigorous audit template
- `references/lean-paper-verification-checklist.md` — 6-point checklist for verifying paper proofs against Lean (boundary indices, commuting pairs, chain bridge, parameter counts)
- `references/mathematical-writing-principles.md` — Tao, Halmos, AMS writing principles
- `references/paper-imitation-workflow.md` — imitation workflow
- `references/referee-attack-checklist.md` — 7-round attack template
- `references/compression-examples.md` — Lean→paper compression examples
- `references/nullspace-audit.md` — 6-step nullspace verification method
- `references/constraint-provenance-audit.md` — constraint matrix rank audit
- `references/paper-lean-proof-consistency-audit.md` — proof-level Lean correspondence
- `references/foundation-audit.md` — Lean definition vs math definition audit
- `references/parameter-lifecycle-audit.md` — variable lifecycle trace audit
- `references/mechanism-and-blind-referee-audit.md` — mechanism + blind referee audit
- `references/paper-architecture-audit.md` — global → narrative → cognitive → dependency audit
- `references/post-freeze-editing-criterion.md` — classification guide
- `references/markdown-formatting.md` — LaTeX pitfalls in markdown
- `references/referee-rigor-ladder.md` — finalization-stage hardening

## Sentence-Level Writing DNA (from reference paper analysis)

When rewriting a paper to match published derivation-classification conventions,
load `mathematical-writing` and consult `references/reference-paper-analysis.md`
for the sentence-level writing DNA extracted from Ou-Wang-Yao (2007, LAA) and
Kaygorodov-Khrypchenko (2023, arXiv). Covers:

- Two computational philosophies (subspace mod-reasoning vs entry computation)
- Ou's 6-step bracket calculation template
- KK's entry computation template with intermediate equation numbering
- KK's "Combining... we see that..." merge pattern
- Construction template following Ou §2 (A)-(E) format
- Forbidden words list (Therefore, Hence, vanishes, contradiction, induction...)
- Narrative arc (3-section structure, no Conclusion)
- Sentence rhythm rules (median 17 words, alternating declarative/procedural)

The HalfDer(N_n) project has additional write-up resources at:
- `/workspace/rewrite-plan.md` — complete section-by-section plan
- `/workspace/forensic-step1.md` — line-by-line forensic of Ou Step 1

- `references/audit-checklist.md` — 4-dimension rigorous audit template
- `references/mathematical-writing-principles.md` — Tao, Halmos, AMS writing principles
- `references/paper-imitation-workflow.md` — imitation workflow
- `references/referee-attack-checklist.md` — 7-round attack template
- `references/compression-examples.md` — Lean→paper compression examples
- `references/nullspace-audit.md` — 6-step nullspace verification method
- `references/constraint-provenance-audit.md` — constraint matrix rank audit
- `references/paper-lean-proof-consistency-audit.md` — proof-level Lean correspondence
- `references/foundation-audit.md` — Lean definition vs math definition audit
- `references/parameter-lifecycle-audit.md` — variable lifecycle trace audit
- `references/mechanism-and-blind-referee-audit.md` — mechanism + blind referee audit
- `references/paper-architecture-audit.md` — global → narrative → cognitive → dependency audit
- `references/post-freeze-editing-criterion.md` — classification guide
- `references/markdown-formatting.md` — LaTeX pitfalls in markdown
- `references/referee-rigor-ladder.md` — finalization-stage hardening
- `references/reference-paper-templates.md` — structural templates from published derivation-classification papers (Ou-Wang-Yao 2007, Kaygorodov-Khrypchenko 2023)
