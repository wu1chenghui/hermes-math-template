# C-fire Local 2×2 Closure (width-2 k=2)

When `width_c_chain_zero` descends to source `(2,4)` with target `(2,n)`,
the split at k=3 produces child `coeffOf(3,4;3,n) = V(3)`. This creates a
2×2 linear system that closes WITHOUT hI, IH, or any external lemma.

## The three equations

```
  (1) coeffOf_cond at (2,4;2,n), k=3:  2·c = V
  (2) half-Leibniz T2 family at V(3):  V = 2·b
  (3) coeffOf_cond at (1,4;1,n), k=2:  2·b = c
```

where `c = coeffOf(2,4;2,n)`, `b = coeffOf(1,4;1,n)`, `V = coeffOf(3,4;3,n)`.

## Resolution (char≠2)

From (1)+(2): `2c = 2b → c = b`. From (3): `2b = c`. Substituting: `2b = b → b = 0 → c = 0 → V = 0`.

All in `F` with `CharNeTwo`, using only `mul_right_cancel₀` and `ring`.

## Implementation pattern

```lean
-- Eq1: coeffOf_cond at (2,4;2,n), k=3
have hcond24 := coeffOf_cond_of D 2 4 2 n ... 3 ...
-- Expand: only C fires (2=2, 3<n) → 2·coeffOf(2,4;2,n) = coeffOf(3,4;3,n)
have hA24 : (if 2<3 ∧ n=4 then ... else 0) = 0 := by apply if_neg; omega
have hB24 : (if 2=3 ∧ 4<n then ... else 0) = 0 := by apply if_neg; omega
have hD24 : (if 2<2 ∧ n=3 then ... else 0) = 0 := by apply if_neg; omega
have hC24_true : (if 2=2 ∧ 3<n then coeffOf D 3 4 3 n else 0) = coeffOf D 3 4 3 n := by
  apply if_pos; exact ⟨rfl, by omega⟩
rw [hA24, hB24, hC24_true, hD24] at hcond24
-- hcond24: 2·coeffOf(2,4;2,n) = coeffOf(3,4;3,n)

-- Eq2: V(3) = 2·coeffOf(1,4;1,n) via half-Leibniz T2 family
-- bracketIdentity(3,4, 1,3, 1,n) expands to -coeff(3,4;3,n) [only T2 fires]
-- bracketSource(3,4, 1,3, 1,n) = -coeff(1,4;1,n)
-- half-Leibniz: 2·(-coeff(1,4;1,n)) = -coeff(3,4;3,n) → V = 2·b
have hhl := D.half_leibniz 3 4 1 3 1 n ...
have hbs_val : bracketSource D.coeff 3 4 1 3 1 n = - D.coeff 1 4 1 n := by
  unfold bracketSource; simp [show (4:ℕ)≠1 by omega]
have hbi_val : bracketIdentity D.coeff 3 4 1 3 1 n = - D.coeff 3 4 3 n := by
  rw [bracketIdentity_eq_expanded]
  simp [show (1:ℕ)≠3 by omega, show n≠4 by omega, show 3<n by omega]
rw [hbs_val, hbi_val] at hhl
-- hhl: 2·(-b) = -V → negate both sides → V = 2·b
have htemp := congrArg (fun (t : F) => -t) hhl
simp at htemp  -- -(2*(-b)) = -(-V) → 2*b = V
-- V = 2·b, convert to coeffOf
have hV3_coeffOf : coeffOf D 3 4 3 n = 2 * coeffOf D 1 4 1 n := ...
  -- via coeffOf_f for both sides

-- Eq3: coeffOf_cond at (1,4;1,n), k=2 → 2·b = c
have hcond14 := coeffOf_cond_of D 1 4 1 n ... 2 ...
-- Expand: only C fires (1=1, 2<n) → 2·coeffOf(1,4;1,n) = coeffOf(2,4;2,n)
-- (same if_pos/if_neg pattern)

-- Combine:
rw [hV3_coeffOf] at hcond24
-- hcond24: 2·c = 2·b
have h_eq24_14 : coeffOf D 2 4 2 n = coeffOf D 1 4 1 n := by
  have h2 : (2:F)≠0 := CharNeTwo.char_ne_two
  apply mul_right_cancel₀ h2
  simpa [mul_comm] using hcond24
-- From Eq3: 2·b = c
rw [h_eq24_14] at hcond14
-- hcond14: 2·b = b → (2-1)·b = 0 → b = 0
have hz : coeffOf D 1 4 1 n = 0 := by
  have htemp2 : 2 * coeffOf D 1 4 1 n = coeffOf D 1 4 1 n := by simpa using hcond14
  have h_one : (1:F) * coeffOf D 1 4 1 n = 0 := by
    calc 1*x = (2-1)*x := by ring
      _ = 2*x - x := by ring
      _ = x - x := by rw [htemp2]
      _ = 0 := by ring
  simpa using h_one
rw [h_eq24_14, hz]
```

## Pitfalls

- Use `if_pos`/`if_neg` with `omega`, NOT `split_ifs` or `simp [h]` for ∧-conditions.
  `simp [h]` where `h:¬(A∧B)` leaves residual `A→B→` implications.
- Use `congrArg (fun t => -t) hhl` + `simp` to solve `2·(-x) = -y`, NOT `calc` + `ring`.
  `ring` on field expressions with `-` can produce spurious equations.
- `mul_right_cancel₀ h2` cancels on the RIGHT — use `simpa [mul_comm]` to adjust.
- Use `linarith`-free algebra (pure `ring` + `rw`) since `F` is an arbitrary `Field`.

## When to use

- Any `width_c_chain_zero` width-2 branch where `k=2` (source row 2).
- The σ-mirror `width_a_chain_zero` has an analogous 2×2 for boundary-right.
- General pattern: when TWO coeffOf_cond expansions + ONE half-Leibniz produce
  a closed linear system over `F` with `CharNeTwo`.
