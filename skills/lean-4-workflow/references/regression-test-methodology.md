# Regression Test Methodology for Lean Refactoring

When refactoring a Lean project's foundation (e.g., replacing axioms, changing structure types),
use this layered regression approach.

## Step 1: Don't modify — import and audit

Import old modules alongside new ones. Count:
- `Pass`: theorems that compile as-is against the new foundation
- `Need rewrite`: theorems that need mechanical changes (import paths, binder names)
- `Need redesign`: theorems that genuinely fail under the new axioms

## Step 2: Categorize failures

### Type A — Import path changes
Old imports like `open HalfDerivation` become `open Coefficient`.
Fix: mechanical rename.

### Type B — Proof shorter
Old proofs used complex chains (compatibility → bridge → pair → only).
New proofs are single-function calls (e.g., `coeffOf_nonadjacent`).
Fix: rewrite with the shorter proof.

### Type C — Genuine failure
The new axioms don't support the old proof strategy.
Fix: redesign the proof for the new foundation.

## Step 3: Dependency audit

Count usage of key lemmas in the new Applications layer:

```
bracket_zero_coeff : N uses  ← Projection corollary
coeffOf_cond       : N uses  ← Projection corollary
compatibility      : 0 uses  ← SHOULD BE ZERO
half_deriv_cond    : 0 uses  ← SHOULD BE ZERO
linarith           : 0 uses  ← SHOULD BE ZERO
```

If `compatibility = 0` and `half_deriv_cond = 0`, the old architecture is fully eliminated.

## Step 4: Proof complexity audit

Compare old vs new proof sizes:

| theorem | old lines | new lines |
|---------|-----------|-----------|
| bracket_zero_coeff | 204 (sorry) | 1 |
| only_A | ~30 (compatibility chain) | ~15 (bracket_zero_coeff projection) |
| bridge_left | ~20 (chain) | 2 (achain_eq) |

If average proof length decreases, the new foundation is both correct AND more natural.

## Step 5: Architecture alignment

Verify the final dependency graph is strictly acyclic and matches the paper:

```
Matrix → Bracket → Derivation → Projection → Coefficient → Applications → Classification
```

Requirements:
- No reverse imports
- No `compatibility`
- No `HalfDerivation`
- No factor 2
- `coeff` always derived from Projection
