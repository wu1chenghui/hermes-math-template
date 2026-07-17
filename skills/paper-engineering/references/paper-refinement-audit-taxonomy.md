# Paper Refinement Audit Taxonomy

> Derived from the HalfDer classification paper refinement (2026-07-02/03).
> Defines the sequence and purpose of each audit stage. Ordered by when to apply.

## Audit Sequence

### 0. Project Charter (before any audit)
Establish immutable constraints before touching the paper:
- Status (Research/Architecture/Proof/Writing)
- Frozen Contracts (which sections cannot be modified)
- Success Criterion (when is the paper done)
- Abandoned Approaches (what not to revisit)
- Lean/Paper relationship (reference implementation vs narrative)

### 1. Global Architecture Audit
**Question**: Does each section have a unique, necessary role?
- Check section dependency DAG (no cycles)
- Check forward references
- Check information flow (definition → use order)
- Identify sections that can be merged or deleted

### 2. Logical Compression Audit
**Question**: Can anything be deleted without losing mathematical content?
- Delete orphaned corollaries (never referenced)
- Compress verbose proofs
- Remove duplicated explanations
- Rename "design document" titles to "paper" titles

### 3. Narrative Audit
**Question**: Does the reader know WHY they are reading each section?
- Add motivation sentences at section/subsection starts
- Add cognitive flow bridges between sections
- Add ending summaries ("this completes the characterization of...")

### 4. Cognitive Load Audit
**Question**: Does the reader need to hold too much in working memory?
- Check definition-use distance (recall if >2 pages)
- Check new-concept density per page
- Add narrative pauses between dense lemma sequences

### 5. Dependency Audit
**Question**: What does the main theorem actually depend on?
- Build theorem dependency DAG
- Identify orphans, single-use items, mergeable pairs
- Verify no hidden circularities

### 6. Boundary Audit
**Question**: Are edge cases explicitly verified?
- Check smallest valid n
- Check endpoint behavior
- Verify degenerate cases (e.g., n=4 when theorem states n≥5)

### 7. Hostile Referee Audit
**Question**: Can a hostile reader find a genuine mathematical gap?
- "Clearly/obviously" scan
- "Therefore/hence" step-size check
- Quantifier completeness
- Every "by Lemma X" → does Lemma X actually prove this?

### 8. Constraint Provenance Audit
**Question**: Which constraints ACTUALLY eliminate which degrees of freedom?
- Build full constraint matrix for a small n
- Compute marginal rank contribution of each constraint class
- Compare with paper's claimed mechanism
- **Key finding from HalfDer**: self-pair claimed n-1 DOFs, actual contribution ~1

### 9. Proof Architecture (FREEZE)
- Define constraint layers L0–L7 ordered by LOGICAL DEPENDENCY
- Map each layer to Lean theorem
- Define Writing Discipline rules
- FREEZE — no further structural changes

### 10. Mechanism Audit
**Question**: Does each subsection do exactly one thing?
- Check for forward references WITHIN a section
- Check for parameter counting in mechanism sections
- Check for basis names in mechanism sections

### 11. Blind Referee Audit
**Question**: Can a first-time reader follow the entire paper?
- Read PDF only, no project context
- 7-round test: Introduction, Object, Motivation, Proof, Dependency, Story, Interruptions

### 12. Paper↔Lean Consistency Audit (Proof-Level)
**Question**: Does every paper proof step have a Lean counterpart?
- Trace proof chains, not just theorem names
- Check for "paper claims X, Lean proves Y" mismatches
- Verify dependency DAGs align

### 13. Notation Sweep
**Question**: Are any symbols ambiguous across sections?
- Check for a_k (coordinate) vs a-channel (notation) conflicts
- Rename using Greek letters for coordinate labels

## Key Principles

1. **Audit before edit.** Never rewrite a section until the audit confirms WHAT needs to change.
2. **Freeze before writing.** Architecture must be stable before sections are rewritten.
3. **Add, don't delete.** Prefer inserting bridge sentences over removing existing content.
4. **Verify with computation.** When a mechanism claim is suspect, build the matrix and compute the rank.
5. **Lean is the oracle.** If paper and Lean disagree, Lean wins. Fix the paper.
