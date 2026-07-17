# D3-SKELETON Implementation Patterns (sharp Lean-encoding lessons)

From D3-SKELETON-0/1/2A (2026-06-25), capturing the patterns that
cost iterations during the frozen-dispatch, incremental-fill phase.

## 1. `set` vs `obtain` for definitional equality

```lean
-- WRONG: `k` is only propositionally `i+1`; `rfl` fails
obtain ⟨k, hik, hkj⟩ : ∃ k, i < k ∧ k < j := ⟨i+1, by omega, by omega⟩
have hk_ip1 : k = i+1 := rfl        -- ❌ type mismatch

-- RIGHT: `k` is definitionally `i+1`
set k := i+1 with hk_ip1
have hik : i < k := by omega
have hkj : k < j := by omega
-- then `rw [hk_ip1]` works everywhere
```

**Rule**: use `set ... with` when the witness value must be used in rewrites.
Use `obtain` only when the exact witness value is irrelevant.

## 2. `bracket_zero` argument order

`D.bracket_zero i j c d u v` expects exactly **11 proof arguments**:

| pos | meaning | typical |
|-----|---------|---------|
| 1 | 1 ≤ i | `hi` |
| 2 | i < j | `by omega` |
| 3 | j ≤ n | `hjn` |
| 4 | 1 ≤ c | `by omega` |
| 5 | c < d | `by omega` |
| 6 | d ≤ n | `by omega` |
| 7 | 1 ≤ u | `by omega` |
| 8 | u < v | `by omega` |
| 9 | v ≤ n | `by omega` |
| 10 | **commuting**: i+1 ≠ c | `hcom1` |
| 11 | **commuting**: i ≠ d | `hcom2` |

Total = 9 validity + 2 commuting conditions. Mistaking this order
causes "expected `v+1 ≤ n` but got `i+1 ≠ v`" style errors.

## 3. `subst hv_n` conflicts with `variable {n}`

When `n` appears both as a `variable {n : ℕ}` binder AND is the
substitution target, `subst hv_n` replaces `v` with `n` but then
`n` references in the body clash. Prefer `rw [hv_n]` instead.

## 4. Nat subtraction arithmetic — avoid omega on `n-1`

`omega` frequently fails on terms containing `n-1` (Nat subtraction
truncation).  Use explicit arithmetic:

```lean
have hi1_eq_n1 : i+1 = n-1 := Nat.le_antisymm hi1_le_n1 (Nat.le_of_not_lt hi1_lt_n1)
have hi2_eq_n : i+2 = n :=
  calc i+2 = (i+1)+1 := by omega
    _ = (n-1)+1 := by rw [hi1_eq_n1]
    _ = n := Nat.sub_add_cancel (by omega)
```

## 5. `simpa` after `coeffOf_f` rewrite beats `linear_combination`

```lean
-- bracketIdentity expansion gives hb : coeff(i,j;u,v) - 0 + 0 - 0 = 0
rw [coeffOf_f D i j u v ...]   -- goal: D.coeff i j u v = 0
simpa using hb                 -- closes goal (simplifies -0+0-0)
-- `linear_combination hb` fails because it can't handle the -0 terms
```

## 6. `by_contra!` on `v < n` produces `n ≤ v`

`by_contra! H` pushes `¬(v < n)` to `H : n ≤ v` (not `H : ¬ v < n`).
Use `Nat.le_antisymm hvn H` to get `v = n`.

## 7. Child-∉I closure — use WidthFiltration lemmas

For the nonadjacent recursion's 4 child target targets, use:
- `not_I_target_of_row_ge_3 hn r c h3r` — row ≥ 3 ⇒ ∉ I
- `not_I_target_of_col_lt_n1 hn r c hc` — col < n-1 ⇒ ∉ I
- `not_I_target_row2_col_ne_n hn c hc` — row=2, col≠n ⇒ ∉ I

These are in `WidthFiltration` — open it with `open WidthFiltration`
in the file-wide `open` block.

## 9. `Nat.decreasingInduction` — dependent motive (mathlib4)

When descending from `n` to `i` on a predicate that needs the `ℓ ≤ n` proof:

```lean
-- The motive takes BOTH the index AND a proof that the index ≤ n
let P : (ℓ : ℕ) → (ℓ ≤ n) → Prop := fun ℓ _ =>
  ∀ (j' : ℕ), ℓ + 2 ≤ j' → j' ≤ n → coeffOf D ℓ j' 2 n = 0

-- Base: P n (le_refl n) — usually vacuously true
have h_base : P n (le_refl n) := by
  intro j' hn2 hn'; exfalso; omega

-- Step: for k < n, P(k+1) → P(k)
-- NOTE: the ih's second argument is `h : k < n` (= k.succ ≤ n),
-- which matches P(k+1)'s second argument exactly.
have h_step : ∀ (k : ℕ), (h : k < n) → P (k + 1) h → P k (Nat.le_of_lt h) := by
  intro k hk ih_plus j' hkj' hj'n
  -- split at k+1, use ih_plus where children have larger source row
  ...

-- Apply: relies on explicit (motive := P) since P is a `let` binder
have hi_le_n : i ≤ n := by omega
have h_res : P i hi_le_n :=
  Nat.decreasingInduction (motive := P) h_step h_base hi_le_n
exact h_res j hij hjn
```

**Critical gotchas**:
- `P` must take TWO arguments `(ℓ : ℕ) → (ℓ ≤ n) → Prop`, NOT just `ℕ → Prop`.
  The non-dependent version fails with "expected type is not available."
- `of_succ`'s third argument has type `P (k+1) h` where `h : k < n`.
  This works because `k < n` = `k.succ ≤ n` definitionally in mathlib4.
- The return type is `P k (Nat.le_of_lt h)` where `Nat.le_of_lt h : k ≤ n`.
- Must use `(motive := P)` explicitly — the elaborator cannot infer the
  motive from a `let` binder.

## 8. ZERO (interior u>i) — two sub-cases

- **V < n**: partner = (V, V+1), T1 extraction, zero residual.
- **V = n, u ≥ i+3**: partner = (i+2, u), T2 extraction, zero residual.
- **V = n, u = i+2**: NOT zero-residual — routes to TRANSFER.

Both sub-cases follow the same skeleton:
```lean
have hb : bracketIdentity ... = 0 := D.bracket_zero ... <11 args>
rw [bracketIdentity_eq_expanded] at hb
-- write 4 c1..c4 conditions (1 `if_pos`, 3 `if_neg`)
rw [if_pos c1, if_neg c2, if_neg c3, if_neg c4] at hb
rw [coeffOf_f D ...]
simpa using hb
```
