# Confluence + Evaluation Proof Pattern (2026-06-19)

Discovered during the HalfDerivation Leaf Theory closure. Used to prove
equalities between adjacent-diagonal coefficients via the confluence theorem.

## Pattern: Confluence on E(i, i+N) with two splits

```lean
have h_conf := eval_diagonal_invariant D i (i+N) k1 k2 hi hij hjn hik1 hk1j hik2 hk2j
-- h_conf: coeff(i,k1;i,k1) + coeff(k1,i+N;k1,i+N)
--       = coeff(i,k2;i,k2) + coeff(k2,i+N;k2,i+N)
```

Then expand the middle diagonal terms via `eval_diagonal`:
```lean
have h_mid1 : 2 * coeffOf D p q p q = coeffOf D p k p k + coeffOf D k q k q :=
  eval_diagonal D p q k hp hpq hqn hpk hkq
```

The algebraic manipulation always uses `ring` and `calc` — NEVER `linarith`
(latter fails on `Field F`).

## Proving 2*x = x → x = 0 (when char ≠ 2)

```lean
have h_zero : x = 0 := by
  have : (2 : F) * x - x = 0 := by rw [h_2x_eq_x, sub_self]
  have h_simp : (2 : F) * x - x = x := by ring
  rw [h_simp] at this
  exact this
```

## Proving 2*x = 2*y → x = y (when char ≠ 2)

```lean
have h_eq : x = y := by
  have hchar : (2 : F) ≠ 0 := CharNeTwo.char_ne_two
  calc
    x = (1/2 : F) * (2 * x) := by field_simp [hchar]
    _ = (1/2 : F) * (2 * y) := by rw [h_2x_eq_2y]
    _ = y := by field_simp [hchar]
```

## Proving a_i1 + M = 2*a_i1 → M = a_i1

```lean
have h_M_eq : M = a_i1 := by
  calc
    M = (a_i1 + M) - a_i1 := by ring
    _ = 2 * a_i1 - a_i1 := by rw [h_temp]
    _ = a_i1 := by ring
```

## set + rw pitfall

`eval_diagonal_invariant` and `eval_diagonal` return raw `coeffOf D ...`
expressions. When using `set` to abbreviate these:

```lean
set a_i := coeffOf D i (i+1) i (i+1) with ha_i
-- but h_conf from eval_diagonal_invariant has raw `coeffOf D i (i+1) i (i+1)`
-- rw [ha_i] at h_conf  -- FAILS: can't match
-- Fix: use rw [← ha_i] at h_conf to unify the set var with raw expr
```

Rule of thumb: use `rw [← hX]` not `rw [hX]` when the hypothesis contains
the raw expression and you want to replace it with the `set` variable.

## skip_two_eq proof structure

Goal: a[i] = a[i+2] where a[k] = coeff(k,k+1;k,k+1)

1. Confluence on E(i,i+3) with splits i+1, i+2:
   a[i] + coeff(i+1,i+3) = coeff(i,i+2) + a[i+2]

2. eval_diagonal for middle terms:
   2*coeff(i+1,i+3) = a[i+1] + a[i+2]
   2*coeff(i,i+2) = a[i] + a[i+1]

3. Algebra: 2*(coeff(i,i+2) - coeff(i+1,i+3)) = a[i] - a[i+2]
   From confluence: coeff(i,i+2) - coeff(i+1,i+3) = a[i] - a[i+2]
   So: 2*(a[i] - a[i+2]) = a[i] - a[i+2] → a[i] = a[i+2]

## adjacent_step_eq proof structure

Goal: a[i] = a[i+1] for i where i+4 ≤ n

1. Confluence on E(i,i+4) with splits i+1, i+2:
   a[i] + coeff(i+1,i+4) = coeff(i,i+2) + coeff(i+2,i+4)

2. eval_diagonal:
   2*coeff(i+1,i+4) = a[i+1] + coeff(i+2,i+4)
   2*coeff(i+2,i+4) = a[i+2] + a[i+3]
   2*coeff(i,i+2) = a[i] + a[i+1]

3. skip_two_eq: a[i+2] = a[i], a[i+3] = a[i+1]

4. Simplify: coeff(i,i+2) = coeff(i+2,i+4) (both = (1/2)(a[i] + a[i+1]))

5. Express coeff(i+1,i+4) = (1/2)(a[i+1] + coeff(i+2,i+4))

6. Plug into confluence, cancel a[i], solve for coeff(i+2,i+4) = a[i+1]

7. Then 2*a[i+1] = a[i] + a[i+1] → a[i] = a[i+1]
