# CE-Generation Lemma vs Skeleton Divergence

> Discovered 2026-06-27 through proof design audit. This documents the fundamental
> architectural mismatch between the paper's image-containment proof and the
> current Lean skeleton.

## The Paper's Proof Structure

The paper proves image containment (D3: HalfDer = F·Id ⊕ kerΦ) through a
**source-width induction** (CE-generation lemma):

```
For any source (i,j) with width w = j-i:
  w ≥ 2: split at i+1
    half-Leibniz expresses coeff(i,j) as linear combination of children's coeffs
    children have smaller width → induction applies
    → wide-source coefficients DETERMINED by adjacent-source coefficients
    → inactive sources (no (1,3)/(n-2,n) in reduction): combination = 0
    → active sources: explicit, all targets in I
  w = 1 (adjacent): base case
    Global classification: CenterMaps(n-1) + BoundaryCocycles(5)
    All targets in I by construction
```

Key: the induction variable is SOURCE WIDTH, not target position.
The paper NEVER discusses "first-row rectangles" — that concept is absent.

## The Skeleton's Divergence

The Lean skeleton (`imageContainment_skeleton`) uses a DIFFERENT induction:
target width `d = v-u`, and for adjacent sources (width=1) does a **per-target
case split** (u>i / u=i / u<i) with separate mechanisms for each case.

This creates artificial difficulties:
- L2099 (coeff(i,i+1;1,v) for u=1, v>i+1): a "rectangular first-row" case
  that is structurally invisible to all local bracketIdentity mechanisms
- In the paper, this case doesn't exist — interior adjacent source (i,i+1)
  is a CenterMap whose only nonzero target is (1,n)

## The CE-generation lemma has NEVER been implemented in Lean

All existing modules (WidthFiltration, Spanning) take `I_filtered` as a
HYPOTHESIS. No module proves Φ=0 → I_filtered. The skeleton tried to prove
it per-coefficient but diverged from the paper's source-width induction.

## Current State

- ImageContainment.lean: ORPHAN module, zero importers
- Live pipeline: RepresentationBridge → Spanning → DimensionTheorem (complete)
- D3 (HalfDer = F·Id ⊕ kerΦ): NOT YET WIRED, needs Φ=0 → I_filtered
- The skeleton's 8 sorries are in a dead-end proof branch

## Diagnostic: Adjacency DAG Reconstruction

Methodology: draw the paper's propagation DAG (frozen ← boundary ← interior),
map every Lean lemma to an edge, identify missing/gap edges.

Result: the DAG is complete for targets near n (col=n, n-1), but has NO edge
for first-row targets with col ≤ n-2. This is a gap in the DAG DESIGN, not
a missing lemma — the boundary mechanism's "frozen landing" only works when
the target column is n (→ (2,n)∈I) or n-1 (→ (1,n-1)∈I for σ-mirror).

## Proof Design Audit (reusable technique)

When per-coefficient proof search stalls:
1. Read the paper's proof structure — what is the INDUCTION on?
2. Reconstruct the paper's propagation DAG
3. Map all existing Lean lemmas to DAG edges
4. Identify which edges are missing vs. which case splits are artifacts

This revealed that the skeleton's case split (u>i/u=i/u<i) is NOT present
in the paper — the paper handles all adjacent sources uniformly through
global classification.
