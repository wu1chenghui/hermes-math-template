# Proof Architecture Patterns

Collected during the 1/2-derivation classification project (June 2026).

---

## Pattern 1: Common Layer Abstraction

**When to use**: Two or more structures share a subset of axioms that produce
identical theorems. Instead of proving the theorems twice, extract the shared
axioms into a minimal interface.

**Example**: NnDerivation (full) and HalfDerivation (1/2) both satisfy
`bracketIdentity = 0` for commuting sources. This is the ONLY property needed
by ONLY and Pair lemmas. Extract it into `ProjectionIdentity`:

```lean4
structure ProjectionIdentity (F : Type) [Field F] where
  n : ℕ
  coeff : ℕ → ℕ → ℕ → ℕ → F
  bracket_zero (i j c d u v : ℕ) ... : bracketIdentity coeff i j c d u v = 0

-- Both structures project to it:
def NnDerivation.toProjectionIdentity (D : NnDerivation F n) : ProjectionIdentity F := ...
def HalfDerivation.toProjectionIdentity (D : HalfDerivation F n) : ProjectionIdentity F := ...

-- Theorems proved ONCE:
theorem only_A (pi : ProjectionIdentity F) ... := ...
theorem pair_cancel_AC (pi : ProjectionIdentity F) ... := ...
```

**Key rule**: Do NOT abstract more than the shared axioms. If a theorem needs
`coeffOf_cond` (which differs between architectures), it does NOT belong in
the common layer. The user's explicit guidance: "不要为了抽象而抽象，要按照真正的
数学依赖来分层" (Don't abstract for abstraction's sake — layer by real
mathematical dependencies).

**Signs you're abstracting too much**:
- The interface needs a parameter that can't be inferred from function types
- A theorem's proof differs between architectures even though the statement is the same
- You're adding "just in case" fields that nothing uses yet

---

## Pattern 2: Reduction Tree Theory

**When to use**: An algebraic system induces a combinatorial transition graph
on index patterns. Extract the graph structure and prove properties about it
INDEPENDENTLY of the algebraic content.

**Example**: coeffOf_cond defines transitions between coefficient indices
(i,j;u,v) → (i',j';u',v') via splits. These transitions:
- Always strictly decrease source width: j'-i' < j-i
- Form a DAG (no cycles)
- Terminate at adjacent sources (width=1)
- Have leaf count 2^(w-2) for diagonal targets

These properties are PURELY COMBINATORIAL — they depend only on integer
inequalities, not on the field F, the derivation structure, or the factor 2.

**Implementation**: Create a separate theory file that proves these properties
using only `Nat` arithmetic:

```lean4
-- E/Reduction/ReductionTree.lean — imports only Mathlib.Tactic, no project modules
def leftChildWidth (i k : ℕ) (hik : i < k) : ℕ := k - i
def rightChildWidth (k j : ℕ) (hkj : k < j) : ℕ := j - k

theorem width_strictly_decreases (i j k : ℕ) (hik : i < k) (hkj : k < j) :
    leftChildWidth i k hik < j - i ∧ rightChildWidth k j hkj < j - i := ...
```

**Value**: This separates "what is true about the index structure" from
"what is true about the coefficients." The index structure is architecture-
independent; the coefficient values differ between full and half derivations.

**Discovering these patterns**: Run experiments FIRST. The ChainSearch and
ReductionTree experiments revealed width monotonicity before any theorem
was proved. Let experiment output guide which theorems to formalize.

---

## Pattern 3: Same Theorem, Different Proof

**When recognized**: Two architectures prove the SAME statement but through
DIFFERENT proofs because one axiom differs by a factor.

**Example**: `coeffOf_nonadjacent` — both NnDerivation and HalfDerivation
prove "coeff = 0 for nonadjacent off-diagonal sources." But:
- NnDerivation uses `coeffOf_cond` → `coeff = A-B+C-D` → direct
- HalfDerivation uses `coeffOf_cond_of` → `2*coeff = A-B+C-D` → factor 2 via CharNeTwo

**Key insight**: The induction step of coeffOf_nonadjacent is purely combinatorial
and IDENTICAL between both versions. Only `coeffOf_source_exists` (the "bootstrap"
lemma) differs. This means the factor 2 only appears ONCE, at the entry point,
and the rest of the proof is architecture-independent.

**Implementation**: Place each version in its own file (`Nonadjacent.lean`,
`HalfNonadjacent.lean`). Do NOT try to force them into a common layer —
they genuinely need different axioms. Document that they prove the same
statement.

---

## Pattern 4: Experiment-Driven Structure Discovery

**Philosophy**: Before proving theorems, write `#eval!` experiments that
explore the combinatorial structure. Let the experiments tell you what is true.

**This session's experiments**:
1. `ProjectionSearch.lean` — classify A/B/C/D active patterns for any (i,j,c,d,u,v)
2. `ChainSearch.lean` — compute the coeffOf_cond transition tree
3. `ReductionTree.lean` — systematic width analysis, invariants, leaf counts

**What experiments revealed**:
- Width strictly decreases at EVERY step (verified across all 110 transitions in N_6)
- All leaves are adjacent sources (width=1)
- Leaf count = 2^(w-2) — perfect binary tree
- A/C chains preserve endpoint alignment
- The structure is a DAG, not a complex graph

**Method**: Run the experiment FIRST, then prove the theorems it suggests.
This avoids wasting time proving things that aren't true, and reveals
patterns that wouldn't be obvious from the axioms alone.

**Self-contained rule**: Experiment files should import ONLY `Mathlib.Tactic`
(no project modules) so they run independently of the build system.
Use `lake env lean --run E/Experiments/Foo.lean`.
