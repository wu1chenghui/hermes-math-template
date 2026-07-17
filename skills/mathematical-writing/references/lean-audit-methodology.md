# Lean-to-Paper Audit Methodology

When a paper has an accompanying Lean formalization, the most reliable way to find gaps in the paper is to compare the proof structure against the Lean proof, lemma by lemma.

## Procedure

1. **Identify the Lean theorem** corresponding to each paper lemma. Use `grep -rn 'theorem\|lemma'` in the Lean project to find relevant files.

2. **Read the Lean proof structure**, not the details. Focus on:
   - What equations are used? (e.g., `Phi coeff i j c d u v = 0`)
   - What auxiliary lemmas are called? (e.g., `F1_coeff_relation`, `adjacent_above_zero`)
   - Is the Lean case breakdown different from the paper's?

3. **For each subcase in the paper**, verify against Lean:
   - Does the paper's partner choice match Lean's `bracketSource = 0` (commuting) or `bracketSource ≠ 0` (non-commuting)?
   - Does the paper's projection target match what Lean's `bracketIdentity` computes?
   - Are boundary conditions handled correctly? (Lean's `omega` solver catches off-by-one errors that papers miss)

4. **Flag discrepancies**:
   - Different case structure → paper may need restructuring
   - Different equations → paper may have wrong computation
   - Missing boundary handling → paper's "for all k≥3" may need to be "for 3≤k≤n-2"

## Common failure modes found

### 1. Over-claiming "for all k≥3" without boundary exception
Lean's `interior_c_zero` uses `3 ≤ p → p ≤ n-2`, not just `p ≥ 3`. The boundary case `p = n-1` gives a different equation (the coupling relation).

### 2. Claiming chain-bridge recurrences without verifying
The paper claimed `2c_{k,k+2} = c_k + c_{k+1}` from the chain bridge. Direct bracket computation showed BOTH terms are zero for interior k. Lean has NO such recurrence — the constraints come from F1/F2 (commuting pair half-Leibniz), not from the chain bridge.

### 3. Wrong projection targets in commuting-partner arguments
Paper: target `(u,v)`, partner `E_{v,v+1}`. Both brackets zero → 0=0, no constraint.
Fix: target `(u, v+1)`. First bracket gives desired coefficient via `[E_{uv}, E_{v,v+1}] = E_{u,v+1}`.

### 4. Using non-commuting pairs with Ψ=0 notation
Paper wrote `Ψ(...)=0` for a pair that doesn't commute. Ψ is `[φx,y] + [x,φy]` (no bracketSource). For non-commuting pairs, the full half-Leibniz `2φ[x,y] = Ψ` must be used. Lean uses `Phi = 2*bracketSource - bracketIdentity = 0` uniformly, which is correct for both commuting and non-commuting pairs.

### 5. Chain bridge applied to wrong source (cross-source conflation)
**Found in Case 1a/1b of image restriction (2026-07-10).** The paper has an equation like `π_{1v}(φ(E_{12})) + π_{2,v+1}(φ(E_{v,v+1})) = 0` and tries to zero the second term by applying the chain bridge to source `(2,v+1)`. But the chain bridge at source `(2,v+1)` gives information about `φ(E_{2,v+1})`, not about `φ(E_{v,v+1})`. These are different sources — the chain bridge relates coefficients of a single source across width reductions, it does not transfer information across different source indices.

**Detection**: When a paper argument says "apply the chain bridge to source (X,Y)" and then concludes something about coefficient `π_{...}(φ(E_{A,B}))` where `(A,B) ≠ (X,Y)`, the reasoning is invalid.

**Fix**: Use `internal_det_coupling_zero` (the 1d three-equation chain-bridge system) instead. In Lean, the correct dispatch for Case 1a/1b is through the `offdiag_zero_v_lt_n` theorem which routes to `internal_det_coupling_zero` for the second-bracket term, never through the chain bridge applied to a different source.

### 6. Forward references that create circular dependencies

**Found in the centered decomposition proof (2026-07-10).** Lemma~1.1 (centered decomposition in §1) references Lemma~3.1 (diagonal propagation in §3). This isn't a mathematical error — the proof is correct — but it creates a structural dependency where a reader encounters an unproven claim. 

**Detection**: For each `\ref{}` in the paper, verify that the label is defined earlier in the document (or at least in the same section for things like equation references). Cross-section forward references to lemmas/theorems are a red flag — they indicate the proof may need reordering.

**Fix**: Reorder sections so that all dependencies flow forward. In this case: chain bridge → width reduction → diagonal propagation → centered decomposition (all now in §1).

| File | Content |
|------|---------|
| `Centering.lean` | `centered_image_in_I`, `adjacent_above_zero`, `adjacent_below_zero`, `boundary_rigidity` |
| `Spanning.lean` | `F1_coeff_relation`, `F2_coeff_relation`, `interior_a_zero`, `interior_c_zero`, `a_channel_normal_form`, `c_channel_normal_form` |
| `DimensionTheorem.lean` | Generator family A_k + B/C/D/E/F, linear independence, finrank |
| `ImageContainment.lean` | `internal_det_coupling_zero` (Case 1d of image restriction) |

### 7. "Blanket sentence" fix for scope precision

**Found in Case 2(iii) of image restriction (2026-07-10).** A subcase references a result ("zero by Case~1") but the result was proved only for adjacent sources while the subcase may involve non-adjacent sources. The naive fix is to add "and width reduction" to every affected reference, but a cleaner approach is to insert a single blanket sentence at the scope boundary:

> By width reduction (Lemma~1.1), the conclusions of Case~1 extend from adjacent sources to all sources: for every (p,q) and every (u,v) with v<n, (u,v)∉I, we have π_{uv}(φ(E_{pq}))=0.

This one sentence at the transition between Case~1 and Case~2 legitimizes every downstream "zero by Case~1" without per-case qualification. **Pattern**: when a result needs scope extension across a proof boundary, do it once at the boundary, not N times at each reference.
