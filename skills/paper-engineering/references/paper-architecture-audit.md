# Paper Architecture Audit

> When a paper draft exists and all proofs are closed, but the paper
> reads like Phase documents. The transition from "correctness audit"
> to "architecture audit."

## When to run

- Proof is complete (0 mathematical blockers).
- Paper draft exists (§1–§8 or equivalent).
- User says "the paper needs to be publishable" or "an independent
  referee should be able to read this."

## What it is NOT

- NOT a correctness audit (lemma-by-lemma verification).
- NOT a local proof audit.
- NOT a search for missing cases.

It is a **structural** audit: does the paper work AS A PAPER?

---

## Priority 0: Logical Compression Audit

> Run AFTER Global Architecture and BEFORE Narrative. Do not try to
> improve the story while the paper still contains redundant material.

### Editing standard

Every paragraph must either **define**, **prove**, or **motivate**.
If a paragraph does none of these, delete it.

### Classification system for compression candidates

| Class | Action | Example |
|-------|--------|---------|
| **A** | Delete or move immediately | Repeated summary subsection, Lean remark in mathematical body |
| **B** | Compress but keep | Proof that is too long but substantively needed |
| **C** | Keep (possibly rename) | Commutative diagram, motivation section |

### Objects that should SURVIVE compression

- **Commutative diagrams / cognitive maps.** They don't carry proof steps
  but help readers navigate. Keep.
- **Motivation sections.** Rename from "Design Principles" to "Motivation
  for the choice of basis" — explaining WHY is rare and valuable. Keep.
- **Object bridges** (e.g., $\ker L = \ker\widetilde L$). These connect
  layers of the architecture. Compress but never delete.
- **Two-level definitions** (e.g., $\VEq \to \SEq$). If they carry the
  object hierarchy, keep both levels.

### Objects that should be REMOVED

- Summary subsections whose content is already in the preceding proof.
- Lean/code remarks in the mathematical body (→ move to appendix).
- Taxonomic classifications never used in actual proofs.
- Pre-theorem padding ("We now prove...", "The following lemma...").

### Lemma inlining rule

If a lemma is: cited exactly once, has a ≤3-line proof, and is only a
stepping stone in the main theorem — inline it into the proof body.

---

## Priority 1: Global Architecture Audit

Check the paper as a whole.

### Questions to answer

1. **Section necessity.** If §N were deleted, would the main theorem
   still be provable? If yes, §N is expository scaffolding — keep it
   light or merge it.
2. **Information flow.** Is every definition used later? Is every
   lemma invoked exactly where needed? Are there forward references
   (a lemma in §N using a result proved in §N+1)?
3. **Minimal dependency path.** Trace the shortest route from
   Foundations (§2) to Main Theorem (§7). Every section NOT on that
   path is supporting structure and should be as light as possible.

### Common findings

- **Phase-document sections.** Sections that read like design notebook
  entries (candidate basis construction, column classification tables,
  "we will determine m in §7"). Compress or absorb into the main proof.
- **Forward references.** A section uses a result proved in a later
  section. Fix by either (a) moving the proof earlier, or (b) marking
  the claim as "will be proved in §X" and NOT using its consequences
  in the earlier section.
- **Triple-announcement.** The same dimension count appears in §3
  (lower bound), §6 (column count), and §7 (main theorem). Pick one
  home for the definitive statement.

---

## Priority 2: Narrative Audit

Check that the reader has a reason to keep reading.

### Questions per section

- §2: Why are these objects being defined?
- §3: Why study HalfDer first?
- §4: Why classify SEq generators?
- §5: Why is propagation the key mechanism?
- §6: Why does the evaluation matrix appear here?
- §7: Why does everything before this force the result?

A section that cannot answer its "why" question needs narrative
restructuring — not more mathematics.

### Reader Walkthrough method

The most effective technique for Narrative Audit: role-play as a
first-time reader. For each section, answer exactly three questions:

1. **What do I now know?** (summary of new information)
2. **What do I most want to ask?** (the reader's natural next question)
3. **Does the next section answer that question?**

If the answer to #3 is "no", the narrative flow has a gap. The fix
is usually a single transition sentence — not a new section.

### Motivation-sentence rule

**Every section and subsection's first paragraph must answer "why
should I read this?"** — not just "what is this?"

- ❌ "The generators are organized by the source-partner relationship."
- ✅ "The subsequent propagation analysis treats each family differently;
  we therefore classify the generators by source-partner geometry."

- ❌ "Currying Ψ in the first two arguments gives a bilinear map..."
- ✅ "To extract scalar constraints from Ψ, we first pass through an
  intermediate vector-valued layer."

If a motivation sentence is missing, add one — don't restructure.

### Suspense control (information pacing)

Don't reveal the importance of an object before the reader feels the
need for it. For example:
- §4 should NOT say "F_eq is the most important generator."
- Instead, let the reader wonder why one T₃ generator survives.
- §5 then reveals the answer.

---

---

## Priority 2.5: Cognitive Load Audit

> Run AFTER Narrative Audit and BEFORE Dependency Audit. The question is
> not "why does this section exist?" but "how much is the reader asked
> to hold in working memory at once?"

### Six dimensions to check per page/section

1. **New concept density.** How many genuinely new mathematical objects
   appear per page? If >3-4 per page, consider splitting or deferring.

2. **New symbol density.** How many new symbols (a_i, b_i, c_i, φ_i,
   T_i, F_eq) appear in one span? If >5-6, some should be deferred.

3. **Working memory span.** When a concept is defined on page N and
   first used on page N+4, the reader has forgotten it. Add a recall
   sentence at the point of first use: "Recall from §2.1 that I = ..."

4. **Definition-use distance.** Ideally ≤ 1-2 pages. If longer, insert
   a recall.

5. **Consecutive lemma fatigue.** If >3 lemmas/corollaries appear in
   sequence with no narrative pause, insert a transition paragraph:
   "The results so far eliminate all interior generators. We now turn
   to the endpoint case, which requires a different argument."

6. **Formula density.** Three consecutive display equations with no
   natural-language bridge → add one sentence connecting them.

### Quick fixes

| Problem | Fix |
|---------|-----|
| Concept defined 4 pages before use | One recall sentence at point of use |
| 7 consecutive lemmas | One narrative pause paragraph |
| Subscript-heavy computation with no context | One sentence explaining WHY before showing subscripts |
| Section with 3 simultaneous concept introductions | Split into smaller subsections |

---

## Priority 2.7: Dependency Audit

> Run AFTER Cognitive Load Audit. Map every labeled item (definition,
> lemma, corollary, theorem) to its references. The goal is to find
> orphaned or single-use items that add proof-tree height without value.

### Method

1. Extract all `\label{...}` from the paper.
2. For each label, find all `\ref{...}` or `\eqref{...}` that cite it.
3. Classify: ORPHAN (0 refs), LOCAL-ONLY (refs only within its own
   section), SINGLE-USE (1 external ref), MULTI-USE (2+ external refs).

### Action rules

| Class | Action |
|-------|--------|
| ORPHAN corollary of another theorem | Delete (it's an immediate consequence, adds no value) |
| ORPHAN named/defined term used only once | Rename (replace with plain language, e.g., "syzygy"→"linear dependency relations" if only used once) |
| SINGLE-USE lemma with ≤3-line proof | Consider inlining into its caller |
| SINGLE-USE lemma with substantive proof | Keep |
| MULTI-USE | Keep |

### Example findings

- `cor:lowerbound` (dim ≥ n+5): 0 refs, immediate consequence of
  `thm:independence`. → Delete.
- `cor:T1redundant`: 0 refs, immediate consequence of `lem:width`. → Delete.
- "syzygy" term: appears 1 time. → Expand to "linear dependency
  relations (syzygies)" on first use, keep term.

### The main theorem dependency DAG

The dependency audit culminates in a clean DAG:
```
§2  eq:bracket ──────────────────────────┐
§3  thm:membership + thm:independence ──┤
§4  channel notation (textual) ──────────┤
§5  lem:chainbridge → lem:width ────┐    │
    lem:diagonal ───────────────────┤    │
    lem:Feq ────────────────────────┼────┤
                                    ▼    ▼
                                 §7 Main Theorem
```
The ideal paper has exactly this structure: the main theorem directly
depends on only 5-6 labeled items. Anything not on this graph is
supporting material — keep it light.

---

## Priority 3: Hostile Referee Audit (Attack Categories)

> Run AFTER Dependency Audit. Read every sentence as a referee trying
> to reject the paper. The following categories are attack vectors.

### Category 1: Hand-wavy expressions

Scan for: `clearly`, `obviously`, `immediately`, `directly`, `routine`,
`it follows`, `one sees`, `one checks`, `one verifies`.

For EACH occurrence, ask: "Can a referee demand the missing step?"
If yes, either add the step or replace with a precise reference.

### Category 2: Therefore/hence/thus chains

For each `therefore`/`hence`/`thus`/`consequently`, verify the gap
between the previous and next statement is exactly ONE logical step.
Multi-step jumps hidden behind a single "therefore" are referee magnets.

### Category 3: Quantifier precision

For each `∀`, `∃`, `for all`, `for every`, `without loss of generality`:
- Is the domain explicit?
- Are hidden conditions (n≥4, char≠2,3) stated?
- Is "without loss" actually justified?

### Category 4: Notation lifetime

Map every symbol's first and last appearance. Flag:
- Symbols defined then unused for ≥3 pages (→ add recall).
- Symbols defined but never used (→ delete).
- Symbols whose definition and use are in different sections with no
  intervening mention (→ add recall or bridge).

### Category 5: Boundary case audit

For every statement involving indices (i,j,k,u,v,n), verify:
- Does it hold at the minimum allowed n (usually n=4)?
- Does it hold at Dynkin endpoints (i=1, j=n, k=n-1)?
- Does it hold when sources are adjacent (j=i+1)?
- Are there any `n≥5` conditions silently assumed in `n≥4` theorems?

**When a suspicious boundary is found**: do NOT adjust the theorem
based on proof-technique failure. First verify the theorem itself
computationally for the small-n case (see
`references/boundary-computation-verification.md`).
Only change the theorem if the computation shows it genuinely differs.
If the theorem holds but the proof needs a separate small-n argument,
add a remark rather than weakening the theorem.

### Category 6: False promises

Check every sentence that says "the next section will..." or "as we
shall see..." — does the promised section actually deliver what was
promised? The most common failure: "determining dim is reduced to
computing rank(M)" when the next section never mentions M.

---

## Priority 4: Independent Readability Audit

Simulate a referee who:
- Has never seen the Phase documents.
- Does not know Lean.
- Was not part of the project.

### Check

Can this reader, using ONLY the paper body (§2–§7), understand:
1. Why SEq is studied?
2. Why propagation matters?
3. Why the candidate basis appears?
4. Why the result is n+5?

If NO, add explanations — not proofs. The problem is exposition, not
missing mathematics.

---

## Priority 4: Appendix (§8) Review

Run AFTER the body is stable. The appendix answers:

> How do Lean and the Paper correspond?

NOT:

> How is the Lean code written?

The correspondence table should map paper theorems to Lean lemmas,
not document the Lean implementation.

---

## Priority 5: Final Consistency Check

- All symbols unique?
- All theorem numbers correct?
- All cross-references resolvable?
- All hypotheses (n≥4, char≠2,3) stated uniformly?
- Terminology ("candidate", "distinguished", "essential") consistent?

---

## What NOT to do during this phase

- Add new mathematical objects.
- Design new Phases.
- Derive new propagation formulas.
- Modify the Lean architecture.
- Do local proof audits (unless the global audit reveals a genuine gap).

These are already complete. The task is exposition, not mathematics.

---

## Key patterns that work

### Fixing a forward reference

```latex
% BEFORE (uses §5 result in §4):
For centered T, Lemma 5 implies Im(T) ⊆ I. Then the T3 generators simplify...

% AFTER (marks as forward reference, defers consequences):
A structural result, proved in Section 5 (Lemma 5), shows Im(T) ⊆ I.
An analysis of the consequences is deferred to Section 5.5.
```

### Compressing a Phase-document section

```latex
% BEFORE (85 lines): B_cand construction, zero-pattern table,
%   column classification, "m is determined in §7"

% AFTER (~50 lines): why the evaluation matrix exists,
%   its definition, its structural role.
```

### Rewriting a pointer-collection proof

```latex
% BEFORE:
By Lemma 5... By Section 6... By Column classification...

% AFTER:
Step 1: Lower bound (from §3).
Step 2: Reduction to adjacent sources (chain bridge).
Step 3: Constraining the channel variables (self-pair + cross-source + F_eq).
Step 4: Counting free parameters → n+5.
```

### The "add, don't delete" rule for project documentation

When updating AGENTS.md or similar chronicle files: NEVER replace
existing content. ALWAYS insert new sections. Existing session
history, documentation indices, and theorem statements are archival
— they enable future agents to trace the project's evolution.
Deletion breaks that trace.
