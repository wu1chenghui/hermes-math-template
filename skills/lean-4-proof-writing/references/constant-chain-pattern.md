# Constant Chain Proof Pattern — Lean 4

Developed during the 1/2-derivation classification project (June 2026).
Proves that all coefficients from a single source `E(i,i+N)` with different
splits are equal (or related by a diagonal sum).

## The pattern

For source `E(i,i+N)` with `N ≥ 2`, every split `k` (`i < k < i+N`) gives
a `coeffOf_cond` expansion:
```
(2:F) * coeff(i,i+N; u,v) = A(k) - B(k) - D(k) + C(k)
```

Since the LHS is independent of `k`, for any two splits `k₁`, `k₂`:
```
A(k₁) - B(k₁) - D(k₁) + C(k₁) = A(k₂) - B(k₂) - D(k₂) + C(k₂)
```

When only ONE term type is active across all splits (e.g., only A-terms),
this simplifies to a constant chain: `A(k₁) = A(k₂)`.

## The three constant chain types

### 1. A-chain (`u < i, v = i+N`)
Only A-terms fire. For any splits `k₁`, `k₂`:
```lean4
theorem achain_lt_i (D : HalfDerivation F n) (i N u : ℕ)
    (hi : 1 ≤ i) (hN : 1 ≤ N) (hiNn : i+N ≤ n)
    (hu : 1 ≤ u) (hu_lt_i : u < i) :
    ∀ (k : ℕ), i < k → k < i+N →
    coeffOf D i k u k = (2 : F) * coeffOf D i (i+N) u (i+N) := by
  -- Use coeffOf_cond. A-term is active (v=i+N ∧ u<k, both true).
  -- B, D, C are all inactive. Simplify with explicit `have` blocks.
```

Corollary: `coeff(i,k₁;u,k₁) = coeff(i,k₂;u,k₂)` for all `k₁`,`k₂`.

### 2. C-chain (`u = i, v > i+N`)
Only C-terms fire. Symmetric structure:
```lean4
theorem cchain_gt_iN (D : HalfDerivation F n) (i N v : ℕ)
    (hi : 1 ≤ i) (hN : 1 ≤ N) (hiNn : i+N ≤ n)
    (hv : 1 ≤ v) (hv_gt_iN : i+N < v) (hvn : v ≤ n) :
    ∀ (k : ℕ), i < k → k < i+N →
    coeffOf D k (i+N) k v = (2 : F) * coeffOf D i (i+N) i v := by
  -- Only C-term fires (u=i ∧ k<v, both true).
  -- Explicitly prove A,B,D are false via `have` blocks, then `simp`.
```

### 3. Diagonal sum (`u = i, v = i+N`)
Both A and C fire:
```lean4
theorem diag_sum_constant (D : HalfDerivation F n) (i N : ℕ)
    (hi : 1 ≤ i) (hN : 1 ≤ N) (hiNn : i+N ≤ n) :
    ∀ (k : ℕ), i < k → k < i+N →
    coeffOf D i k i k + coeffOf D k (i+N) k (i+N) =
    (2 : F) * coeffOf D i (i+N) i (i+N) := by
  -- Both A and C fire. After `simp` with explicit conditions:
  -- hsplit: 2*coeff(i,i+N;i,i+N) = coeff(i,k;i,k) + coeff(k,i+N;k,i+N)
  -- Goal is the reverse: rw [← hsplit]
```

## Proof technique: explicit condition blocks

The key to making these proofs reliable on `Field F` (where `linarith` fails):

```lean4
-- DON'T use split_ifs (16 branches for 4 nested ifs)
-- DON'T use simp with `show ... from by omega` (omega fails in simp context)
-- DO use explicit `have` blocks:

have h_A_false : ¬ (v = i+N ∧ u < k) := by
  intro h; rcases h with ⟨hveq, _⟩; omega
have h_B_false : ¬ (i = k ∧ i+N < v) := by
  intro h; rcases h with ⟨heq, _⟩; omega
have h_C_true : i = i ∧ k < v := ⟨rfl, h_k_lt_v⟩
simp [h_A_false, h_B_false, h_C_true] at hsplit
-- Now hsplit is simplified to the desired equality
```

## Why this pattern is fundamental

The constant chain is NOT just an experimental observation — it's a theorem
that follows directly from `coeffOf_cond`. It's the foundation on which
Bridge, ONLY, and eventually AdjacentBridge are built.

The three types (A-chain, C-chain, Diagonal-sum) are exhaustive: every
`coeffOf_cond` term has one of these forms. The ONLY-type vanishing
(long single-coefficient forced to zero) comes from a different mechanism
(`coeffOf_cond` + `coeffOf_nonadjacent` on specific projection patterns).

## Dependencies

- `Infrastructure.lean` (HalfDerivation, MatIdx)
- `Bracket.lean` (coeffOf, coeffOf_cond, coeffOf_nonadjacent)
- No dependency on Semantics, BracketCondZero, or classification modules
