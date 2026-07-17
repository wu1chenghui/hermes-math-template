# Project Architecture (2026-06-19 final)

## Dual Architecture + Common Layer

The project has THREE layers sharing MatrixBasis + BracketFormula:

### Layer 0: Common Infrastructure (ProjectionIdentity)

```
        ProjectionIdentity (structure)
        · coeff, bracket_zero
        · coeffOf, coeffOf_f
       ┌──────────┴──────────┐
       ↓                     ↓
  NnDerivation          HalfDerivation
  (toProjectionId)      (toProjectionId)
```

ONLY and Pair (8 theorems) are proved at the ProjectionIdentity level — one proof, shared by both theories.

### Layer 1: Reduction Pipeline (HalfDerivation)

```
ReductionTree (pure combinatorics) → Evaluation (algebraic) → Leaf/Adjacent
```

- **ReductionTree**: width strictly decreases, termination, DAG, leaf = adjacent source
- **Evaluation**: diagonal formula 2·root = left+right, CONFLUENCE, edge weight 1/2
- **Leaf/Adjacent**: skip_two_eq — coeff(i,i+1;i,i+1) = coeff(i+2,i+3;i+2,i+3)

### Layer 2: NnDerivation Applications

```
Nonadjacent → Chain → Bridge → Diagonal → Classification
```

Uses coeffOf_cond (no factor 2).

## Full File Map

```
E.lean

E/Core/
├── MatrixBasis.lean           # MatIdx, bracket, CharNeTwo
├── BracketFormula.lean        # bracketLeft/Right/Identity/Source
├── ProjectionIdentity.lean    # COMMON INTERFACE (structure + coeffOf)
├── Derivation.lean            # NnDerivation + toProjectionIdentity
├── HalfDerivation.lean        # HalfDerivation + toProjectionIdentity
├── Projection.lean            # coeffOf wrapper (NnDerivation)
├── HalfProjection.lean        # coeffOf wrapper (HalfDerivation)
└── Coefficient.lean           # coeffOf_cond (NnDerivation)

E/Reduction/
├── ReductionTree.lean         # PURELY COMBINATORIAL (imports only Mathlib.Tactic)
└── Evaluation.lean            # Diagonal formula, Confluence, edge weights

E/Leaf/
└── Adjacent.lean              # skip_two_eq (Confluence + ring algebra)

E/Applications/
├── Only.lean                  # SHARED: only_A/B/C/D (ProjectionIdentity)
├── Pair.lean                  # SHARED: pair_cancel_AC/AD/BC/BD (ProjectionIdentity)
├── Nonadjacent.lean           # NnDerivation: coeffOf_nonadjacent
├── HalfNonadjacent.lean       # HalfDerivation: coeffOf_nonadjacent
├── Chain.lean                 # NnDerivation: achain/cchain/diag_sum
├── Bridge.lean                # NnDerivation: bridge_left/right
└── Diagonal.lean              # NnDerivation: diag_split

E/Experiments/
├── ProjectionSearch.lean
├── ChainSearch.lean
└── ReductionTree.lean         # Experiment version (self-contained)

E/Classification.lean          # Assembly stub
```

## Build

```bash
cd /opt/lean-home/lean-projects/e
source ~/.elan/env
lake build                  # 2967 jobs ✅ (2026-06-19)
```

### Sorry Audit
- Reduction → Evaluation → Leaf → Classification main line: **0 sorry's**
- HalfNonadjacent adjacent boundary: 1 (unreachable — caller requires `2 ≤ j-i`)
- NnDerivation legacy: 3 (unreachable — separate import tree)
- Full ARCHITECTURE.md with DAG + theorem index + statement audit at project root

## Theorem Count

| Layer | Files | Theorems |
|-------|-------|----------|
| Common (ProjectionIdentity) | 1 | 8 |
| Reduction Tree | 1 | 4 |
| Evaluation | 1 | 5 |
| Leaf | 1 | 1 |
| HalfDerivation Core | 3 | 5 |
| NnDerivation Apps | 6 | 19 |
| **Total** | **16** | **42** |

## Dependency DAG

```
Mathlib.Tactic ──────────────────────┐
    │                                 │
MatrixBasis ←── pure ──→ ReductionTree│
    │                                 │
BracketFormula                        │
    │                                 │
ProjectionIdentity (Common)           │
   ┌┴────────────┐                    │
   ↓              ↓                   │
NnDerivation  HalfDerivation          │
   │              │                   │
   │         ┌────┴────┬──────────┐   │
   │         ↓         ↓          ↓   │
   │     HalfProj  HalfNonadj  Eval   │
   │                          │      │
   │                      Adjacent   │
   │                                 │
   └──────→ Applications ────────────┘
```

No circular dependencies. ReductionTree is pure combinatorics.
Evaluation depends only on HalfDerivation + ReductionTree.
Leaf/Adjacent depends on Evaluation.
