# Confluence & Reduction Proof Patterns (2026-06-19)

Patterns discovered during formalization of the Reduction Tree and
Evaluation Theory for the 1/2-derivation classification project.

---

## Pattern 1: Confluence via `eval_diagonal_invariant`

Key identity: for any two splits k₁, k₂ of source E(i,j) (diagonal target),
coeff(i,k₁;i,k₁) + coeff(k₁,j;k₁,j) = coeff(i,k₂;i,k₂) + coeff(k₂,j;k₂,j).

Usage:
```lean4
have h_conf := eval_diagonal_invariant D i j k₁ k₂ hi hij hjn hik₁ hk₁j hik₂ hk₂j
```

Then expand middle terms and cancel:
```lean4
  have h_diff : 2*(Y-X) = A - B := by
    calc
      2*(Y-X) = 2*Y - 2*X := by ring
      _ = (A+C) - (C+B) := by rw [h_Y, h_X]
      _ = A - B := by ring
```

Pitfall: after `field_simp`, need `ring` to close numeric arithmetic.

---

## Pattern 2: `set` variable trap with external hypotheses

PROBLEM: `set M := coeffOf ...` then `rw` on h_conf from an external lemma fails
because h_conf uses raw `coeffOf D ...`, not `M`.

FIX (a): Don't use `set` for expressions in external lemma results.
FIX (b): Use `rw [← ha_i]` to convert raw form to abbreviation first:
```lean4
  rw [← ha_i] at h_conf'
  have h_temp := congrArg (fun x => x - a_i) h_conf'
  simp [sub_self, add_sub_cancel_right] at h_temp
```

---

## Pattern 3: `2*x = x → x = 0` without linarith

linarith fails on Field F. Use ring:
```lean4
  have h_zero : X = 0 := by
    have : (2 : F) * X - X = 0 := by rw [h_diff2, sub_self]
    have h_simp : (2 : F) * X - X = X := by ring
    rw [h_simp] at this; exact this
```

---

## Pattern 4: Induction with `eval_nonadjacent_expand`

For `all_diag_equal`:
```lean4
  induction' hw : j - i using Nat.strong_induction_on with w ih generalizing i j
  by_cases h_adj : j - i = 1
  · exact scalarCoeff_spec D hn5 i hi (by omega)
  · have h_nonadj : 2 ≤ j - i := by omega
    have h_expand := eval_nonadjacent_expand D i j hi hij hjn h_nonadj
    have h_right : coeffOf D (i+1) j (i+1) j = scalarCoeff D :=
      ih (j-(i+1)) (by omega) (i+1) j (by omega) (by omega) hjn rfl
    rw [h_expand, h_left, h_right]
    field_simp [CharNeTwo.char_ne_two]; ring
```

Key: `field_simp` then `ring` for (1/2)*λ + (1/2)*λ = λ.
Use `omega` directly, not `rw [hw]; omega`.
