# Proof Architecture Patterns

Discovered during audit/fix cycles on derivation-classification paper (2026-07-10).
These are cross-cutting patterns that emerge when comparing a paper proof against
a Lean formalization and restructuring for clarity.

## Pattern 1: Subcase Extraction as Claim

**Symptoms**: A dense subcase (20+ lines, multiple equations) is embedded deep in a
case analysis, but is consumed as a dependency by earlier subcases. This creates
a forward-reference problem.

**Example**: Case 1d (three-equation chain bridge system for u<i, v=i+1) is
needed by Case 1a and 1b to zero their second-bracket terms. But 1d comes after
1a and 1b in the enumeration.

**Fix**: Extract the subcase as a standalone Claim BEFORE the case analysis begins.
Give it a self-contained proof. Then:
- The earlier subcases (1a, 1b) reference the Claim with one line
- The original subcase (1d) becomes "This is the Claim."
- No forward references within the proof

**Why this beats reordering**: Reordering subcases (1d before 1a) fixes the immediate
dependency but degrades the logical grouping (subcases ordered by relation between u
and i). The Claim extraction preserves the case-analysis structure while resolving the
dependency. It also signals to the reader that this subcase is load-bearing — it's NOT
just another item in a list, it's a lemma.

## Pattern 2: Blanket Sentence Before Next Case

**Symptoms**: Case A proves a result for adjacent sources. Case B references that
result for a non-adjacent source, requiring width reduction as an implicit bridge.
The paper writes "zero by Case A" but the reasoning is "zero by Case A + width reduction."

**Example**: Case 1 proves coefficients vanish for all adjacent sources with v<n.
Case 2(iii) references Case 1 for source (2,u) which may be non-adjacent (u>3).

**Fix**: Add ONE blanket sentence between Case A and Case B:
  "By width reduction (Lemma X), the conclusions of Case A extend from adjacent
   sources to all sources: for every (p,q) and every target meeting the conditions
   of Case A, the coefficient is zero."
Then every reference to Case A from Case B is valid as-is — no per-case qualifications.

**Anti-pattern to avoid**: Patching each individual reference with "and width reduction."
This clutters the proof and creates maintenance burden.

## Pattern 3: Corollary Tautology Check

**Symptoms**: A Corollary states "For [property P], [definition of P]."
The proof invokes the definition as the key step — circular.
The Corollary was created to package a genuine theorem (e.g., "all diagonal entries
are equal") together with a definition ("centered"), but the packaging obscures
that the Corollary IS the definition plus one application of the theorem.

**Example**:
  Theorem: All adjacent diagonal entries of a 1/2-derivation are equal (=d).
  Definition: A 1/2-derivation is "centered" if all its diagonal entries are zero.
  Corollary: For a centered φ, all diagonal entries are zero.
  Proof: "Since φ is centered, π₁₂(φ(E₁₂))=0 [definition]. By the theorem,
  all diagonal entries equal π₁₂(...)=0."

This is P→P with a detour through the theorem. The theorem is already used in
the centered decomposition lemma, which handles both the d=0 and d≠0 cases.

**Fix**: DELETE the Corollary. The theorem and the definition are already present.
Any reference to the Corollary should cite the theorem + definition (or the
centered decomposition lemma) instead.

**Detection rule**: If a Corollary's conclusion appears verbatim in a Definition
within the same section, it's a tautology. Delete it.

## Pattern 4: Detail-Level Consistency

**Symptoms**: Within a single proof, some subcases have 20-line bracket expansions
while others of COMPARABLE complexity have 1-line claims ("Expanding the brackets
yields...").

**Example**: Case 2(i) (three Ψ-equations) gets an itemize describing each bracket
contribution. Case 2(v) (two pairs, one non-commuting, two-equation system) gets
one sentence.

**Fix**: Match the detail level of the MOST detailed comparable subcase. For Case 2(v),
this means identifying each bracket term (bracket-left, bracket-right), which component
of φ contributes, which δ-term fires, and the sign.

**What makes Case 2(v) special**: It's the only subcase with a NON-commuting pair,
requiring the full half-Leibniz equation (2φ[x,y] on the left, not just Ψ=0).
The bracket expansion must track:
- Bracket-left of first pair: δ_{b,i} and δ_{i+1,a} routes
- Bracket-right of first pair: δ_{u,a} and δ_{b,i} routes
- LHS computation for second pair: -2φ(E_{i,u}) → -2X
- Bracket-left of second pair: both δ routes give 0 at target
- Bracket-right of second pair: δ_{u,a} route gives the surviving term

Each of these needs a sentence explaining which component contributes and why.
