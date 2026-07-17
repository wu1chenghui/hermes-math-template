# D3 Chain Induction — `Nat.decreasingInduction` with Dependent Motive

Pattern for descending source-row chain induction in the D3 `width_c_chain_zero`
family of lemmas (`ImageContainment.lean`). Validated 2026-06-26.

## When to use

When you need to prove a property `P(i)` for all source rows `i` from `n` down to
some lower bound, where the descent step goes from `P(i+1)` to `P(i)`.

## The `Nat.decreasingInduction` signature (mathlib4 v4.31.0-rc1)

```lean4
Nat.decreasingInduction {n : ℕ} {motive : (m : ℕ) → m ≤ n → Sort u}
  (of_succ : (k : ℕ) → (h : k < n) → motive (k + 1) h → motive k (Nat.le_of_lt h))
  (self : motive n (le_refl n))
  {m : ℕ} (mn : m ≤ n) : motive m mn
```

Key points:
- `motive` takes TWO arguments: the index `m` AND a proof that `m ≤ n`.
- `of_succ` receives `h : k < n` (not `k+1 ≤ n`). This works because `k < n`
  is definitionally `k.succ ≤ n` = `k+1 ≤ n`.
- `self` is the base case at `n` (usually vacuously true).
- Returns `motive m mn` — a dependent pair of the result and the `≤ n` proof.

## Recipe: defining P with dependent motive

```lean4
let P : (ℓ : ℕ) → (ℓ ≤ n) → Prop := fun ℓ _ =>
  ∀ (j' : ℕ), ℓ + 2 ≤ j' → j' ≤ n → coeffOf D ℓ j' 2 n = 0

have h_base : P n (le_refl n) := by
  intro j' hn2 hn'; exfalso; omega  -- n+2 ≤ j' ≤ n impossible

have h_step : ∀ (k : ℕ), (h : k < n) → P (k + 1) h → P k (Nat.le_of_lt h) := by
  intro k hk ih_plus j' hkj' hj'n
  -- k is the current source row
  -- ih_plus : P (k+1) h  =  ∀ j', (k+1)+2 ≤ j' ≤ n → coeffOf D (k+1) j' 2 n = 0
  -- Goal: coeffOf D k j' 2 n = 0
  -- Split at m = k+1 via coeffOf_cond_of, use ih_plus for children
  ...

have hi_le_n : i ≤ n := by omega
have h_res : P i hi_le_n :=
  Nat.decreasingInduction (motive := P) h_step h_base hi_le_n
exact h_res j hij hjn
```

## Pitfalls

1. **`Nat.decreasingInduction` elaborator needs explicit `motive`** — old code
   used `Nat.decreasingInduction h_step h_base hi_le_n` but this fails with
   "failed to elaborate eliminator, expected type is not available". Always
   provide `(motive := P)`.

2. **`Nat.le_of_lt` not `Nat.le_of_lt_succ`** — the return type of `of_succ`
   uses `Nat.le_of_lt h` (which gives `k ≤ n` from `k < n`). Do NOT use
   `Nat.le_of_lt_succ` (that's for `m < n.succ → m ≤ n`).

3. **`let` not `set`** — define P with `let` to avoid `set` binder issues
   with the dependent type.

4. **`h : k < n` used as `k+1 ≤ n`** — in `P (k+1) h`, the second argument
   `h : k < n` serves as proof of `k+1 ≤ n` because `k < n` is `k.succ ≤ n`.

## Verified instance

`width_c_chain_zero` in `E/Classification/ImageContainment.lean` (2026-06-26,
B1 checkpoint). The skeleton compiles with `Nat.decreasingInduction (motive := P)`.
