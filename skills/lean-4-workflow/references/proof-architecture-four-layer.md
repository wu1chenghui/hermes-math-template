# Lean 4 Proof Architecture — The Four-Layer Pattern

## When to Use This Pattern

When a Lean project develops circular dependencies between "infrastructure" lemmas
(coefficient calculus) and "classification" lemmas (main theorems), the fix is NOT
to patch individual lemmas but to insert a **semantics bridge layer**.

## The Four Layers

```
Layer 0: Infrastructure  —  MatIdx, HalfDerivation, bracket definitions
Layer 1: Bracket          —  coeffOf, coeffOf_cond, coeffOf_nonadjacent
Layer 2: Semantics        —  bracketLeft/bracketRight, bridge lemmas
Layer 3: Classification   —  ToNCoeff, Adjacent, ImageInCenter, main theorem
```

Layer 2 (Semantics) is the critical insertion. It expresses operator-level identities
as coefficient-level equations, bridging the gap between pure combinatorics (Layer 1)
and classification (Layer 3).

## The Anti-Pattern: Circular Dependencies

```
Bracket.lean  ←  needs adjacent-source lemmas
    ↓
ToNCoeff.lean
    ↓
Adjacent.lean  →  provides those lemmas but through NCoeff.cond_zero
    ↑                              which depends on Bracket.lean
Bracket.lean  ←  (circular)
```

## The Fix

1. Prove what you CAN in Bracket.lean (non-adjacent sources)
2. Create a Semantics bridge that defines `bracketLeft`/`bracketRight`
3. Create BracketCondZero.lean that imports both and closes the gap
4. The downstream chain (ToNCoeff → Adjacent → Classification) imports BracketCondZero

Result: strictly unidirectional dependency graph.

## Key Files to Extract

- `E/HalfDerivation/Semantics.lean` — bridge definitions (no classification imports)
- `E/BracketCondZero.lean` — refactored coeffOf_cond_zero (imports Semantics + Bracket)
- `E/HalfDerivation/AdjacentBridge.lean` — isolated proof of remaining gap
- `E/Length3Analysis.lean` — [2026-06-15] computational enumeration of Length 3 compatibility;
  see `lean-4-proof-writing / references/length3-compatibility-analysis.md` for the
  technique and the inductive bridge finding

## Alternative Strategy: Length 3 Compatibility (Inductive Bridge)

As of 2026-06-15, a computational enumeration revealed that `AdjacentBridge.lean`'s
8 sub-lemmas are NOT independent projections of a single compatibility identity.
Instead, **Length 3 compatibility connects adjacent source coefficients to
intermediate source coefficients**, forming an inductive structure:

```
Adjacent source E(i,i+1) coefficients
        ↕ (Length 3 compatibility)
Intermediate sources: E(i,i+2), E(i+1,i+3) — HAVE splits
        ↕ (coeffOf_cond applicable)
Known vanishing lemmas
```

The PAIR equations from the compatibility are:
```
coeffOf D i (i+1) u (i+1) = coeffOf D i (i+2) u (i+2)
coeffOf D (i+1)(i+3) (i+1) v = coeffOf D (i+2)(i+3) (i+2) v
```

This is a **2-step strategy** (not a 8-lemma grind):
1. Prove the Length 3 compatibility identity (already done: `length3_compatibility_raw`)
2. Project to coefficient equations and close inductively via intermediate sources

See `lean-4-proof-writing / references/length3-compatibility-analysis.md` for the
full computational enumeration results and proof strategy.

## Pitfall: `split_ifs with hA hB` Creates Function Types

`split_ifs with hA hB` gives `hA : P → False` for false branches (not `¬P`).
`rcases` fails on function types. Use bare `split_ifs` with `rename_i` instead.
