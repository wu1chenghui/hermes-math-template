# Projection Analysis — Enumeration-First Methodology

## The Lesson

After multiple failed attempts to directly prove `AdjacentBridge` lemmas,
systematic `#eval!` experiments discovered the true structure: that
single-source compatibility produces Bridge chains (not the pairwise
cancellation the 8-lemma model assumed).

**Rule**: Run enumeration experiments BEFORE writing proofs for new patterns.

## The Framework

`ProjectionAnalysis.lean` enumerates all (u,v) projections for source E(i,i+N)
and classifies the active coefficient pattern per projection:

- Which `coeffOf_cond` terms are active at each split k
- Net coefficient equation after cancellation
- Bridge chains (A-chain, C-chain) connecting different source lengths
- Pattern classification: ZERO / ONLY / PAIR(≠) / MULTI(n)

## Key Findings (across Lengths 4-7)

1. **PAIR(=) disappears after Length 3.** Length 3 is the only case where
   coefficient pairs cancel (opposite signs). For N≥4, all active terms in
   a projection have the same sign.

2. **PAIR(≠) is always exactly 2** regardless of source length. These come
   from the transition boundary of A and C cascades.

3. **Bridge chains are global.** For ALL-A pattern (u<i+1, v=i+N):
   ```
   coeff(i,i+1; u,i+1) = coeff(i,i+2; u,i+2) = coeff(i,i+3; u,i+3) = ...
   ```
   This connects adjacent (L1) to ALL longer source lengths.

4. **ONLY patterns are precisely the D_k, B_k, C_1, and A_{N-1} edge cases.**

## How to Run

```bash
cd /opt/lean-home/lean-projects/e
source ~/.elan/env
lake env lean --run E/ProjectionAnalysis.lean
```

Edit the `#eval!` lines at the bottom to change i or N.

## Pattern: Explicit have hl/hr Blocks (No Fragile Macros)

When projecting from `length3_compat` (or any compatibility identity),
the original `simpl8` macro was fragile. The robust replacement:

```lean4
lemma bridge_left (D : HalfDerivation F n) (i u v : ℕ) ... :
    coeffOf D i (i+1) u (i+1) = coeffOf D i (i+2) u (i+2) := by
  have h_compat := length3_compat D i u v ...
  rw [hv_eq] at h_compat
  -- Compute what the left side simplifies to
  have hl : <big-if-expression> = coeffOf D i (i+1) u (i+1) := by
    simp [hu_lt, hu_ne_i, show ... from by omega, ...]
  -- Compute what the right side simplifies to
  have hr : <big-if-expression> = coeffOf D i (i+2) u (i+2) := by
    have hu_lt_ip2 : u < i+2 := by omega
    simp [hu_lt_ip2, hu_ne_i, show ... from by omega, ...]
  rw [hl, hr] at h_compat
  exact h_compat
```

**Key rules for the `simp` block:**
- Pass ALL needed `≠` hypotheses explicitly (`hu_ne_i`, `show i+3 ≠ i+1 from by omega`)
- Pass ALL needed `¬` comparisons (`show ¬ (i+3 < i+3) from by omega`)
- Use `omega` to generate the hypotheses, don't guess
- Better to have too many `simp` hypotheses than too few (extra ones are harmless)
