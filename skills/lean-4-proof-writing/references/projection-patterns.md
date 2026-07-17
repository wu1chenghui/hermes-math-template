# Projection-Centered Proof Patterns (2026-06-18)

These patterns emerged from the refactoring that replaced `HalfDerivation`
(coeff primitive + half_deriv_cond axiom) with `NnDerivation` (Leibniz rule).
All lemmas become corollaries of Projection — no compatibility, no factor 2.

## Pattern 1: bracket_zero_coeff → ONLY or Pair lemma

For commuting sources E(i,j) and E(c,d) (j≠c, i≠d), Leibniz gives:
  `bracketIdentity coeff i j c d u v = 0`

To prove `only_A` (only term A active):
  1. Call `bracket_zero_coeff` → `bracketIdentity = 0`
  2. Prove 3 inactive terms = 0 using explicit `have` blocks with `simp [h_false]`
  3. Conclude active term = 0
  4. Convert `D.coeff` → `coeffOf` via `coeffOf_f`

Key: use FOUR explicit `have` blocks (one per if-condition), not `split_ifs`.

```lean4
  have hA_val : (if u < c ∧ v = d then D.coeff i (i+1) u c else 0) = D.coeff i (i+1) u c := by
    simp [huc, hvd]
  have hB_zero : (if u = c ∧ d < v then D.coeff i (i+1) d v else 0) = 0 := by
    simp [show u ≠ c from by omega]
  have hC_zero : (if u = i ∧ i+1 < v then D.coeff c d (i+1) v else 0) = 0 := by
    simp [hC_false]
  have hD_zero : (if u < i ∧ v = i+1 then D.coeff c d u i else 0) = 0 := by
    simp [hD_false]
  unfold bracketIdentity bracketLeft bracketRight at h_bracket
  simp [hA_val, hB_zero, hC_zero, hD_zero] at h_bracket
  -- h_bracket is now: (active term) = 0
```

## Pattern 2: coeffOf_cond → Chain lemma

For source E(i,i+N) with N≥2, `coeffOf_cond` with split k gives
the chain equality directly. NO factor 2 needed.

A-chain (u < i target): all B/C/D terms vanish, only A remains.
C-chain (v > i+N target): all A/B/D terms vanish, only C remains.
Diagonal (u=i, v=i+N): A and C terms survive.

```lean4
theorem achain_eq (D : NnDerivation F n) (i N u k : ℕ) ... :
    coeffOf D i k u k = coeffOf D i (i+N) u (i+N) := by
  have h_cond := coeffOf_cond D i (i+N) u (i+N) ... k hik hk_iN
  simp [huk, hu_ne_k, hu_ne_i, show i+N ≠ k from by omega, ...] at h_cond
  exact h_cond.symm
```

## Pattern 3: simpa [sub_eq_add_neg]

Bracket expansion gives `A + (-B) = 0` but goal is `A - B = 0`.
`exact h_bracket` fails with type mismatch. Fix:

```lean4
  simpa [sub_eq_add_neg] using h_bracket
```

Or include it in the main `simpa`:
```lean4
  simpa [sub_eq_add_neg, hcoeff1, hcoeff2] using h_bracket
```

## Pattern 4: D.coeff vs coeffOf conversion timing

`bracket_zero_coeff` works with raw `D.coeff`. Downstream theorems
use `coeffOf`. Convert at the END, not in the middle:

```lean4
  -- Work with D.coeff throughout:
  unfold bracketIdentity ... at h_bracket
  simp [...] at h_bracket
  -- h_bracket is in D.coeff terms. Convert goal at the end:
  simpa [hcoeff1, hcoeff2] using h_bracket
```

`rw [hcoeff]` with `coeffOf_f` may fail due to dot notation (`D.coeffOf`).
Prefer `simpa [coeffOf_f ...]` over `rw`.

## Pattern 5: Chain → Bridge (trivial)

Bridge is a direct corollary of Chain:
  `bridge_left` = `achain_eq` with N=3, k=i+1 vs k=i+2 (2-line proof)
  `bridge_right` = `cchain_eq` with N=3 (2-line proof)

No new machinery needed — just instantiate Chain theorems.

## Pattern 6: Pair lemmas from bracket_zero_coeff

All 4 pair lemmas (AC, AD, BC, BD) are special cases of `bracket_zero_coeff`
where exactly TWO terms are active. Each is ~12 lines.

```lean4
  have h_bracket := bracket_zero_coeff D i (i+1) c d u v ...
  have hA_val : (if u<c∧v=d then ...) = D.coeff i (i+1) u c := by simp [huc, hvd]
  have hB_zero : (if u=c∧d<v then ...) = 0 := by simp [show u≠c from by omega]
  have hC_val : (if u=i∧i+1<v then ...) = D.coeff c d (i+1) v := by simp [hui, hp1v]
  have hD_zero : (if u<i∧v=i+1 then ...) = 0 := by simp [show ¬(u<i) from by omega]
  unfold bracketIdentity bracketLeft bracketRight at h_bracket
  simp [hA_val, hB_zero, hC_val, hD_zero] at h_bracket
  simpa [sub_eq_add_neg, hcoeff1, hcoeff2] using h_bracket
```

## Pattern 7: Diagonal split from coeffOf_cond

Single-step diagonal decomposition `coeff(i,j;i,j) = coeff(i,i+1;i,i+1) + coeff(i+1,j;i+1,j)`
follows from `coeffOf_cond` with split k=i+1, target E(i,j):

```lean4
  have h_cond := coeffOf_cond D i j i j hi hij hjn hi hij hjn (i+1) (by omega) h_nonadj
  simp [show i < i+1 from by omega, show i ≠ i+1 from by omega,
    show ¬(i < i) from by omega, show j ≠ i+1 from by omega, h_nonadj] at h_cond
  exact h_cond
```

## General Principle

Every lemma should be either:
  * `coeffOf_cond` → a Chain/Bridge/Diagonal equality
  * `bracket_zero_coeff` → an ONLY/Pair vanishing

Never use compatibility, half_deriv_cond, factor 2, or linarith on Field F.

---

## Pattern 8: Common Layer Abstraction (2026-06-19)

When two structures (e.g. NnDerivation and HalfDerivation) share a commuting-source
identity (`bracketIdentity = 0` for j≠c, i≠d), extract the shared interface into
a third structure. Parameterize commuting-source theorems against it:

```lean4
structure ProjectionIdentity (F : Type) [Field F] [CharNeTwo F] where
  n : ℕ
  coeff : ℕ → ℕ → ℕ → ℕ → F
  bracket_zero (i j c d u v : ℕ) ... : bracketIdentity coeff i j c d u v = 0

-- Bridge from each concrete structure:
def NnDerivation.toProjectionIdentity (D : NnDerivation F n) : ProjectionIdentity F := ...
def HalfDerivation.toProjectionIdentity (D : HalfDerivation F n) : ProjectionIdentity F := ...

-- Shared theorems: proved ONCE, apply to BOTH
theorem only_A (pi : ProjectionIdentity F) ... : coeffOf pi i (i+1) u c = 0 := ...
```

**When to use**: Two structures have overlapping axioms for a subset of cases.
The shared theorems only use the overlapping axioms → extract them.

**When NOT to use**: When theorems use axioms that differ between structures
(e.g. `coeffOf_cond` has factor 2 in HalfDerivation but not in NnDerivation).
Those theorems stay architecture-specific.

**Key insight (mathematical)**: The commuting-source calculus (`bracketIdentity=0`
for j≠c, i≠d) is IDENTICAL for full and 1/2 derivations. Therefore ONLY and Pair
(which only need commuting sources) belong to the common layer.

## Pattern 9: Same Theorem, Different Proof (2026-06-19)

When a theorem's STATEMENT is identical across two architectures but its PROOF
differs (because it uses architecture-specific axioms), write separate versions
in separate files. Do NOT try to force them into the common layer.

Example: `coeffOf_nonadjacent` — the theorem is `coeff(i,j;u,v)=0` for j-i≥2.
  * NnDerivation proof: uses `coeffOf_cond` (no factor 2)
  * HalfDerivation proof: uses `coeffOf_cond_of` (factor 2, cancelled via CharNeTwo)
  * The proof structures are parallel but the factor-2 handling in `coeffOf_source_exists` differs

```lean4
-- NnDerivation: E/Applications/Nonadjacent.lean
theorem coeffOf_nonadjacent (D : NnDerivation F n) ... : coeffOf D i j u v = 0 := ...

-- HalfDerivation: E/Applications/HalfNonadjacent.lean
theorem coeffOf_nonadjacent (D : HalfDerivation F n) ... : coeffOf D i j u v = 0 := ...
```

**The induction step is identical** (purely combinatorial). Only the base lemma
(`coeffOf_source_exists`) differs in its factor-2 handling.

## Pattern 10: Confluence + Ring Algebra (skip_two_eq, 2026-06-19)

When a confluence property (invariant across reduction paths) combines with
a structural evaluation formula, use ring algebra in the field to extract
equalities. `linarith` does NOT work on `Field F` — use `ring` instead.

Example: `skip_two_eq` — proves coeff(i,i+1;i,i+1) = coeff(i+2,i+3;i+2,i+3)

Strategy:
  1. Confluence on source E(i,i+3): A + X = Y + B
  2. Diagonal evaluation: 2X = C + B, 2Y = A + C
  3. Ring algebra: 2(Y-X) = (A+C)-(C+B) = A-B = Y-X → Y-X = 0 → A = B

```lean4
  set A := coeffOf D i (i+1) i (i+1) with hA
  set B := coeffOf D (i+2) (i+3) (i+2) (i+3) with hB
  set C := coeffOf D (i+1) (i+2) (i+1) (i+2) with hC
  set X := coeffOf D (i+1) (i+3) (i+1) (i+3) with hX
  set Y := coeffOf D i (i+2) i (i+2) with hY

  -- Step 1: Confluence
  have h_conf := eval_diagonal_invariant D i (i+3) (i+1) (i+2)
    hi hi_lt_i3 hip3n hi_lt_i1 hi1_lt_i3 hi_lt_i2 hi2_lt_i3

  -- Step 2: Diagonal evaluation of middle terms
  have h_X : 2 * X = C + B := eval_diagonal D (i+1) (i+3) (i+2) ...
  have h_Y : 2 * Y = A + C := eval_diagonal D i (i+2) (i+1) ...

  -- Step 3: Ring algebra (NOT linarith!)
  have h_diff1 : Y - X = A - B := by
    calc
      Y - X = (Y + B) - B - X := by ring
      _ = (A + X) - B - X := by rw [← h_conf]
      _ = A - B := by ring

  have h_diff2 : 2 * (Y - X) = A - B := by
    calc
      2 * (Y - X) = 2*Y - 2*X := by ring
      _ = (A + C) - (C + B) := by rw [h_Y, h_X]
      _ = A - B := by ring

  -- From h_diff1 and h_diff2: 2*(Y-X) = Y-X → Y-X = 0
  have h_yx_zero : Y - X = 0 := by
    have : (2 : F) * (Y - X) - (Y - X) = 0 := by rw [h_diff2, h_diff1, sub_self]
    have h_simp : (2 : F) * (Y - X) - (Y - X) = Y - X := by ring
    rw [h_simp] at this; exact this

  -- Finally: Y-X = 0 and Y-X = A-B, so A = B
  rw [h_yx_zero] at h_diff1
  exact eq_of_sub_eq_zero h_diff1.symm
```

**Key pitfall**: The step `2*(Y-X) = Y-X → Y-X = 0` uses the field property
that `2 ≠ 0` (CharNeTwo). In a general ring, `2*x = x` does NOT imply `x = 0`
(e.g., in char 2, `2*x = 0 = x` for any x). The ring calculation `2*x - x = x`
is valid in any ring — the cancellation is `2*x = x → 2*x - x = 0 → x = 0`.

## Pattern 11: Structure field vs theorem binding (2026-06-19)

When a structure has a field `coeff : ℕ → ℕ → ℕ → ℕ → F` and a theorem
`bracket_zero` defined in the same namespace, `D.bracket_zero` tries to
access it as a STRUCTURE FIELD and fails because it's a theorem, not a field.

**Fix**: Use explicit lambda wrapping in `toProjectionIdentity`:

```lean4
def toProjectionIdentity (D : NnDerivation F n) : ProjectionIdentity F where
  n := n
  coeff := D.coeff
  bracket_zero := fun i j c d u v hi hij hjn hc hcd hdn hu huv hvn hj_ne_c hi_ne_d =>
    bracket_zero D i j c d u v hi hij hjn hc hcd hdn hu huv hvn hj_ne_c hi_ne_d
```

NOT `bracket_zero := D.bracket_zero` (fails because bracket_zero is a theorem,
not a field — even though it's in the same namespace).
