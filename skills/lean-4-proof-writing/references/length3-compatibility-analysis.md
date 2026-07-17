# Length 3 Compatibility — Inductive Bridge for Adjacent Sources

**Date**: 2026-06-15
**Session**: Investigation of whether AdjacentBridge's 8 lemmas are projections
            of Length 3 split compatibility.

## The Technique

When a Lean proof hits a wall (8 unsolved sub-lemmas all `sorry`), a pure
computational enumeration can reveal hidden structure BEFORE attempting proofs.

### Method

1. Write a **pure computational model** that tracks which terms of an identity
   are active for each target position, using raw `if` conditions (no `D` needed).

2. Compile as a **standalone Lean file** with `lake env lean` (NOT `lean_run_code`
   — the REPL environment lacks `Nat` arithmetic instances).

3. Enumerate all possible index configurations and classify results.

4. Interpret patterns: ZERO (auto-cancel), ONLY (single coefficient = 0),
   PAIR (two coefficients relate), MULTI (degenerate).

### Code Template

```lean4
import Mathlib.Tactic

structure Term where
  name : String
  src  : String
  tgt  : String
  sgn  : Int
  deriving Repr

def allTerms (i u v : ℕ) : List Term := Id.run do
  let mut out : List Term := []
  if <condition> then
    out := {name := "A", src := "i,i+1", tgt := s!"({u},{v})", sgn := 1} :: out
  -- ... enumerate all 8 terms from coeffOf_cond
  return out.reverse

def groupTerms (ts : List Term) : List (String × Int) :=
  -- fold over terms, group by (src → tgt) key, sum signs

#eval! go i u v  -- compile with `lake env lean File.lean`
```

### Pitfalls

- `lean_run_code` fails for `ℕ` comparisons (`LT ℕ`, `HAdd ℕ Nat ℕ` missing).
  Always use `lake env lean File.lean` for standalone computational analysis.
- `List.get?` does not exist in core Lean 4. Use pattern matching instead:
  ```lean4
  match net with
  | [] => "ZERO"
  | [p] => "ONLY"
  | [p1, p2] => "PAIR"
  ```
- `#eval!` (with `!`) needed when ANY error exists in the file, even if
  the function being evaluated is fine. Plain `#eval` aborts on `sorry`.
- `String.intercalate` exists and works for formatting.

## Key Finding (2026-06-15)

### The Context

Project: 1/2-derivation classification on N_n. The sole remaining gap is
`half_deriv_bracket_zero_adjacent` in `AdjacentBridge.lean`, decomposed into
8 sub-lemmas (only_A through pair_BD).

The user's conjecture was that all 8 lemmas are projections of a single
Length 3 compatibility identity from `coeffOf_cond`.

### The Experiment

For source E(i,i+3) (length 3), two splits exist:
- k₁ = i+1: E(i,i+1) + E(i+1,i+3)
- k₂ = i+2: E(i,i+2) + E(i+2,i+3)

`coeffOf_cond` gives an expression for `2·coeffOf D i (i+3) u v` for each split.
Equating them: `RHS₁ = RHS₂`, which simplifies to a compatibility identity with
up to 8 terms (A, B, C, D from RHS₁; A', B', C', D' from RHS₂ with flipped signs).

Enumerating all valid target positions (u,v) for i=5, source E(5,8):

### Results Summary

| Pattern | Count | Description |
|---------|-------|-------------|
| ZERO | 9 | Both splits give zero (auto-cancel) |
| ONLY | 8 | Single coefficient forced to 0 |
| PAIR | 2 patterns × multiple (u,v) | Two coefficients must be equal |
| MULTI(4) | 1 | Target = source itself (degenerate) |

### Which AdjacentBridge Lemmas Appear

**YES, with specific (c,d):**

| Canonical case | AdjacentBridge lemma | Condition |
|---|---|---|
| (u<i, v=i+1) | only_D | c=i+1, d=i+3 |
| (u=i, i+1<v<i+3) | only_C | c=i+1, d=i+3 |
| (u=i+1, v>i+3) | only_B | c=i+1, d=i+3 |

**NO — new patterns not in the 8 lemmas:**

| Case | Source | Target | Equation |
|---|---|---|---|
| u<i, v=i+2 | (i+2,i+3) | (u,i) | coeff = 0 |
| u=i+2, v>i+3 | (i,i+2) | (i+3,v) | coeff = 0 |
| u<i, v=i+3 | (i,i+1) vs (i,i+2) | coeff₁ = coeff₂ (PAIR) |
| u=i, v>i+3 | (i+1,i+3) vs (i+2,i+3) | coeff₁ = coeff₂ (PAIR) |

### The Inductive Bridge (Core Insight)

**Length 3 compatibility does NOT produce all 8 AdjacentBridge lemmas.
It produces something BETTER: an inductive bridge.**

```
Adjacent source E(i,i+1) coefficients
        ↕ (Length 3 compatibility)
Intermediate source coefficients:
  E(i,i+2) — length 2, HAS split at k=i+1
  E(i+1,i+3) — length 2, HAS split at k=i+2
  E(i+2,i+3) — length 1 (adjacent)
        ↕ (coeffOf_cond — applicable!)
Known vanishing lemmas (coeffOf_nonadjacent)
```

The PAIR equations say:
```
coeffOf D i (i+1) u (i+1) = coeffOf D i (i+2) u (i+2)
coeffOf D (i+1)(i+3) (i+1) v = coeffOf D (i+2)(i+3) (i+2) v
```

The ONLY equations give direct vanishing for intermediate sources.
Together, they form a **complete inductive reduction**: adjacent source
identities reduce to intermediate source identities, which ARE tractable.

### Why This Refutes the Original Conjecture (8 lemmas = projections)

The original AdjacentBridge decomposition fixes the second source (c,d) as
arbitrary, and produces 8 different patterns depending on which if-conditions
fire for A, B, C, D.

Length 3 compatibility **does not produce all 8 patterns for arbitrary (c,d)**.
Instead, it produces a **different, more structured** set of identities
that connect the adjacent source to its neighboring intermediate sources.

### Updated Proof Strategy

Old strategy: attack 8 independent sub-lemmas one by one.
New strategy: prove the Length 3 compatibility identity, then project.

This is a 2-step process:
1. **Compatibility theorem**: `RHS₁(u,v) = RHS₂(u,v)` for all (u,v)
   (already proven in `length3_compatibility_raw`)
2. **Projection classification**: For each (u,v) region, simplify the
   identity to a concrete coefficient equation
3. **Inductive closure**: The resulting equations either (a) directly vanish,
   or (b) relate to shorter-length sources where `coeffOf_cond` applies.

### Lean Artifacts

The analysis file lives at:
`/opt/lean-home/lean-projects/e/E/Length3Analysis.lean`

It contains:
- `allTerms`: pure computational model of the 8-term compatibility identity
- `groupTerms`: fold-and-sum to cancel ZERO terms
- `classify`/`summarize`: pattern classification
- Manual test cases covering all canonical (u,v) regions

### Relationship to Semantics Bridge Pattern

This is a **dual** to `references/semantics-bridge-pattern.md`:

- **Semantics Bridge**: operator-level identity → projection → coefficient identity
  (for commuting sources — bracket identity)
- **Length 3 Compatibility**: split compatibility → projection → coefficient identity
  (for single source with multiple splits — HD axiom)

Both patterns replace case-analysis explosions with operator-level reasoning,
but they address DIFFERENT axioms (bracket identity vs. HD splitting).
