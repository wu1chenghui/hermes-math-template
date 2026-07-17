# 3-Equation Det-Coupling Pattern (2×2 Linear System)

Discovered during D3.2 `internal_det_step` implementation (2026-06-26).  This is a
self-contained proof technique that closes an adjacent-source coefficient using
THREE commuting-partner equations forming a 2×2 linear system.

## Problem

Given `HalfDerivation D`, adjacent source `(i,i+1)`, target `(2,i+1)`, prove
`coeffOf D i (i+1) 2 (i+1) = 0`.  i≥3, i+1<n, char≠2.

## Mechanism (three equations, two unknowns)

Let `x = coeff(1,2;1,i)`, `y = coeff(i,i+1;2,i+1)`, `z = coeff(i,n;2,n)`.

| Eq | Source | Partner | Target | Relation |
|----|--------|---------|--------|----------|
| EQ1 | `bracketIdentity(1,2, i,n, 1,n)` | (i,n) | (1,n) | x + z = 0 |
| EQ2 | `bracketIdentity(1,2, i,i+1, 1,i+1)` | (i,i+1) | (1,i+1) | x + y = 0 |
| EQ3 | `coeffOf_cond(i,n;2,n)` split at i+1 | — | (2,n) | 2z = y |

**Commutativity check**: (1,2) commutes with (i,n) since 2≠i (i≥3) and 1≠n.
(1,2) commutes with (i,i+1) since 2≠i and 1≠i+1.

**Algebra**: From EQ1+EQ2: y = z.  From EQ3: 2z = y.  So z = 2z → z = 0 → y = 0.  QED.

## Key insight: why it works in Field F

The closure step `2y = y → y = 0` works in ANY field with char≠2 because:
```
(2-1)·y = 0 → 1·y = 0 → y = 0
```
Use `ring` + explicit `calc`, NOT `linarith` (linarith requires ordered rings).

## Lean implementation template

```lean4
lemma internal_det_step (D : HalfDerivation F n) (hn : 3 ≤ n) (hn5 : 5 ≤ n) (h3 : (3 : F) ≠ 0)
    (i : ℕ) (hi : 3 ≤ i) (hi1_n : i + 1 ≤ n) (hi1_lt_n : i + 1 < n) :
    coeffOf D i (i + 1) 2 (i + 1) = 0 := by
  have hi_lt_n : i < n := by omega
  have hi1_v : 1 ≤ i := by omega
  have h2pos : (2 : ℕ) < i + 1 := by omega
  -- EQ1
  have h_eq1 : coeffOf D 1 2 1 i + coeffOf D i n 2 n = 0 := by
    have hb : bracketIdentity D.coeff 1 2 i n 1 n = 0 :=
      D.bracket_zero 1 2 i n 1 n
        (by omega) (by omega) (by omega) hi1_v hi_lt_n (le_refl n)
        (by omega) (by omega) (le_refl n) (by omega) (by omega)
    rw [bracketIdentity_eq_expanded] at hb
    have c1 : (1 : ℕ) < i ∧ (n : ℕ) = n := ⟨by omega, rfl⟩
    have c2 : ¬((1 : ℕ) = i ∧ (n : ℕ) < n) := by omega
    have c3 : (1 : ℕ) = 1 ∧ (2 : ℕ) < n := ⟨rfl, by omega⟩
    have c4 : ¬((1 : ℕ) < 1 ∧ (n : ℕ) = 2) := by omega
    rw [if_pos c1, if_neg c2, if_pos c3, if_neg c4] at hb
    -- convert D.coeff to coeffOf
    have h1 : D.coeff 1 2 1 i = coeffOf D 1 2 1 i := by
      rw [coeffOf_f D 1 2 1 i (by omega) (by omega) (by omega) (by omega) (by omega) (by omega)]
    have h2' : D.coeff i n 2 n = coeffOf D i n 2 n := by
      rw [coeffOf_f D i n 2 n hi1_v hi_lt_n (le_refl n) (by omega) (by omega) (le_refl n)]
    rw [h1, h2'] at hb
    simpa using hb
  -- EQ2 (same pattern, partner (i,i+1) instead of (i,n))
  have h_eq2 : coeffOf D 1 2 1 i + coeffOf D i (i + 1) 2 (i + 1) = 0 := ...
  -- From EQ1, EQ2: y = z
  have h_xy : coeffOf D i (i + 1) 2 (i + 1) = coeffOf D i n 2 n := by
    calc
      coeffOf D i (i + 1) 2 (i + 1)
          = (coeffOf D 1 2 1 i + coeffOf D i (i + 1) 2 (i + 1)) - coeffOf D 1 2 1 i := by ring
      _ = 0 - coeffOf D 1 2 1 i := by rw [h_eq2]
      _ = -coeffOf D 1 2 1 i := by simp
      _ = 0 - coeffOf D 1 2 1 i := by simp
      _ = (coeffOf D 1 2 1 i + coeffOf D i n 2 n) - coeffOf D 1 2 1 i := by rw [h_eq1]
      _ = coeffOf D i n 2 n := by ring
  -- EQ3
  have h_eq3 : (2 : F) * coeffOf D i n 2 n = coeffOf D i (i + 1) 2 (i + 1) := by
    have hcond := coeffOf_cond_of D i n 2 n hi1_v hi_lt_n (le_refl n) (by omega) (by omega)
      (le_refl n) (i + 1) (by omega) (by omega)
    -- A=if 2<i+1 ∧ n=n, B=if 2=i+1 ∧ n<n (false), C=if 2=i ∧ i+1<n (false), D=if 2<i ∧ n=i+1 (false)
    have hA : (if (2 : ℕ) < i + 1 ∧ (n : ℕ) = n then coeffOf D i (i + 1) 2 (i + 1) else 0)
           = coeffOf D i (i + 1) 2 (i + 1) := by simp [h2pos]
    have hB : (if (2 : ℕ) = i + 1 ∧ (n : ℕ) < n then coeffOf D i (i + 1) n n else 0) = 0 := by
      simp [show (2:ℕ) ≠ i+1 by omega]
    have hC : (if (2 : ℕ) = i ∧ (i + 1 : ℕ) < n then coeffOf D (i + 1) n (i + 1) n else 0) = 0 := by
      simp [show (2:ℕ) ≠ i by omega]
    have hD : (if (2 : ℕ) < i ∧ (n : ℕ) = i + 1 then coeffOf D (i + 1) n 2 i else 0) = 0 := by
      simp [show n ≠ i+1 by omega]
    rw [hA, hB, hC, hD] at hcond
    simpa using hcond
  -- Closure: 2y = y → y = 0
  have hy : coeffOf D i n 2 n = 0 := by
    rw [h_xy] at h_eq3
    have : ((2 : F) - 1) * coeffOf D i n 2 n = 0 := by
      calc
        ((2 : F) - 1) * coeffOf D i n 2 n
            = (2 : F) * coeffOf D i n 2 n - 1 * coeffOf D i n 2 n := by ring
        _ = coeffOf D i n 2 n - coeffOf D i n 2 n := by rw [h_eq3]; simp
        _ = 0 := by ring
    have h_one : ((2 : F) - 1) = (1 : F) := by ring
    rw [h_one] at this
    simpa using this
  rw [h_xy, hy]
```

## Pitfalls

- **`coeffOf_cond_of` guards**: The A-guard is `u < m ∧ v = j` where j is the SOURCE column (= n), v is the TARGET column (= n). For the split at i+1: A = `if 2 < i+1 ∧ n = n then ...`.  Easy to mistake for `i = n` or `i = i`.

- **`Field F` algebra**: Never use `linarith`. Use `ring` + `calc` for field equations.  `2y = y → y = 0` is `(2-1)y = 0 → 1·y = 0 → y = 0`.

- **`rw` on `coeffOf_cond_of` output**: The output uses `coeffOf`, not `D.coeff`. Do NOT `rw [coeffOf_f ...] at hcond` unless you also rewrite the target — the mismatch between `D.coeff` and `coeffOf` will cause `simpa` to fail.

- **Forward reference**: In Lean 4, definitions later in the file can't be referenced earlier. Place `internal_det_step` BEFORE any lemma that calls it.

## Variant: Commuting-Cross SCC (boundary_local_234)

Discovered during BND-L dispatch analysis (2026-06-26 f), implemented 2026-06-26.

### Problem

Three boundary coefficients form a self-contained 3-variable SCC:
`x = coeff(3,n;2,n)`, `y = coeff(3,4;2,4)`, `z = coeff(1,2;1,3)`.
Prove all three are zero. NO hI, NO induction, NO width machinery.

### Mechanism (commuting-cross pattern)

| Eq | Source | Partner | Target | Relation |
|----|--------|---------|--------|----------|
| EQ1 | `bracketIdentity(1,2,3,n,1,n)` | (3,n) | (1,n) | z + x = 0 |
| EQ2 | `bracketIdentity(3,4,1,2,1,4)` | (1,2) | (1,4) | -y - z = 0 |
| EQ3 | `coeffOf_cond(3,n;2,n)` split at 4 | — | (2,n) | 2x = y |

**Key difference from `internal_det_step`**: EQ2 uses **swapped** sources `(3,4)` and
`(1,2)` — the partner `(1,2)` is the FIRST source of EQ1, making this a "cross"
pattern rather than the "bridge" pattern where `(1,2)` is always the first source.
Both sources commute with their partners (2≠3, 1≠n for EQ1; 4≠1, 3≠2 for EQ2).

**Algebra**: From EQ1: z = -x. From EQ2 (after flipping sign): y = x. From EQ3: 2x = x
→ x = 0 → y = z = 0.  det=1, char-independent.

### Sign pitfall with bracketIdentity expansion

EQ2 raw expansion: `0 - y + 0 - z = 0` → `-y - z = 0`.  This is NOT `y + z = 0`.
The `simpa` after `rw [hy, hz] at hb` will fail with a type mismatch unless you
use `simpa [sub_eq_add_neg]` or `simpa [neg_add]` or derive `y + z = 0` separately
with `linear_combination -h_eq2'`.

### Lean implementation (key differences from internal_det_step)

```lean4
lemma boundary_local_234 (D : HalfDerivation F n) (hn5 : 5 ≤ n) :
    coeffOf D 1 2 1 3 = 0 ∧ coeffOf D 3 4 2 4 = 0 ∧ coeffOf D 3 n 2 n = 0 := by
  -- EQ1: z + x = 0 (standard bracket_zero → coeffOf_f → simpa)
  have h_eq1 : coeffOf D 1 2 1 3 + coeffOf D 3 n 2 n = 0 := ...
  -- EQ2: -y - z = 0 (NOTE the minus signs!)
  have h_eq2' : -(coeffOf D 3 4 2 4) - coeffOf D 1 2 1 3 = 0 := ...
  -- Derive positive form for combination step
  have h_eq2 : coeffOf D 3 4 2 4 + coeffOf D 1 2 1 3 = 0 := by
    linear_combination -h_eq2'
  -- EQ3: 2x = y (sole A-term at split k=4, n≠4 from hn5)
  have h_eq3 : (2 : F) * coeffOf D 3 n 2 n = coeffOf D 3 4 2 4 := ...
  -- Combine: x = y (from h_eq1 - h_eq2), then 2x = x → x = 0
  have hx_eq_y : coeffOf D 3 n 2 n = coeffOf D 3 4 2 4 := by
    linear_combination h_eq1 - h_eq2
  have hy0 : coeffOf D 3 4 2 4 = 0 := by
    rw [hx_eq_y] at h_eq3
    -- h_eq3: 2·y = y → solve in Field F
    have htemp : (2 : F) * coeffOf D 3 4 2 4 = (1 : F) * coeffOf D 3 4 2 4 := by
      simpa [one_mul] using h_eq3
    have h_one : ((2 : F) - 1) * coeffOf D 3 4 2 4 = 0 := by
      linear_combination htemp
    simpa [show ((2 : F) - 1) = (1 : F) by ring] using h_one
  -- Wrap up
  have hx : coeffOf D 3 n 2 n = 0 := by rw [hx_eq_y, hy0]
  have hz : coeffOf D 1 2 1 3 = 0 := by rw [hx] at h_eq1; simpa using h_eq1
  exact ⟨hz, hy0, hx⟩
```

### When to use this over the bridge pattern

- **Bridge** (`internal_det_step`): when you can use `(1,2)` as a "bridge" source in
  both bracketIdentity calls. Works for coefficients `coeff(i,i+1;2,i+1)` for i≥3.
- **Cross** (`boundary_local_234`): when the second partner must be `(1,2)` swapped
  with the first source `(3,4)` to get the right surviving terms. Used for BND-L
  boundary coefficients where the partner `(1,2)` commutes with `(3,4)` in both
  directions.

### n≥5 requirement

The `hn5` hypothesis is load-bearing: `n ≠ 4` ensures the D-term of
`coeffOf_cond(3,n;2,n)` at k=4 (`2<3 ∧ n=4`) is false, so only the A-term survives.
Without `n≥5`, the 3-equation system would have a fourth unknown, breaking the SCC.

The failed attempt used `bracketIdentity(i,i+1, i,n, u,i+1)` → T4: `coeff(i,n;u,i) = 0`.
Then `coeffOf_cond(i,n;u,i)` at split i+1.  This fails because the target column `i`
equals the source row `i`, making the A-guard `i = n` (target column = source column),
which is FALSE when i+1 < n.  The T4 approach produces a coefficient at a
*dead-end target* where coeffOf_cond's guards all fail.

**The correct approach uses T1 from `bracketIdentity(1,2, i,n, 1,n)` instead**,
which produces `coeff(i,n;2,n)` — target column `n` = source column `n`, so
the A-guard `n=n` fires.
