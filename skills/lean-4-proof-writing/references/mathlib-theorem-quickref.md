# Mathlib Theorem Quick Reference

Commonly needed mathlib theorems organized by topic, with exact Lean names
and import paths. Use this to find the right theorem without `#check` search.

## Basic Algebra (`import Mathlib.Algebra`)

| Theorem | Type | Notes |
|---------|------|-------|
| `add_comm a b` | `a + b = b + a` | |
| `add_assoc a b c` | `(a + b) + c = a + (b + c)` | |
| `add_zero a` | `a + 0 = a` | |
| `zero_add a` | `0 + a = a` | |
| `add_left_neg a` | `(-a) + a = 0` | Groups |
| `sub_add_cancel a b` | `(a - b) + b = a` | Requires `b ≤ a` in ℕ |
| `add_sub_cancel a b` | `(a + b) - b = a` | |
| `mul_comm a b` | `a * b = b * a` | |
| `mul_assoc a b c` | `(a * b) * c = a * (b * c)` | |
| `mul_one a` | `a * 1 = a` | |
| `one_mul a` | `1 * a = a` | |
| `mul_add a b c` | `a * (b + c) = a*b + a*c` | distributivity |
| `add_mul a b c` | `(a + b) * c = a*c + b*c` | |
| `mul_neg a b` | `(-a) * b = -(a*b)` | Rings |
| `neg_mul a b` | `(-a) * b = -(a*b)` | |
| `sq a` | `a^2` | notation `a^2` |
| `pow_two a` | `a^2 = a*a` | |
| `pow_mul a m n` | `a^(m*n) = (a^m)^n` | |
| `dvd_refl a` | `a ∣ a` | |
| `dvd_trans h1 h2` | `a ∣ b → b ∣ c → a ∣ c` | |

## Natural Numbers (`import Mathlib.Data.Nat.Basic`)

| Theorem | Type | Notes |
|---------|------|-------|
| `Nat.succ_eq_add_one n` | `Nat.succ n = n + 1` | |
| `Nat.add_comm` | ℕ version | |
| `Nat.zero_add` | ℕ version | |
| `Nat.succ_ne_self n` | `Nat.succ n ≠ n` | |
| `Nat.lt_of_lt_of_le` | transitivity of `<` then `≤` | |
| `Nat.le_of_lt` | `a < b → a ≤ b` | |
| `Nat.succ_le_succ` | `a ≤ b → a+1 ≤ b+1` | |
| `Nat.add_succ` | `a + (b+1) = (a+b) + 1` | |
| `Nat.succ_add` | `(a+1) + b = (a+b) + 1` | |
| `Nat.mul_comm` | ℕ version | |
| `Nat.prime_def_lt` | prime definition | |
| `Nat.prime.dvd_of_dvd_mul` | `hp : Prime p` + `p ∣ a*b` → `p ∣ a ∨ p ∣ b` | |
| `Nat.exists_infinite_primes` | `∀ n, ∃ p, n ≤ p ∧ Nat.Prime p` | `import Mathlib.Data.Nat.Prime` |

## Integers (`import Mathlib.Data.Int.Basic`)

| Theorem | Type | Notes |
|---------|------|-------|
| `Int.ofNat_natAbs_of_nonneg` | | |
| `Int.add_comm` | ℤ version | |
| `Int.sub_eq_add_neg` | `a - b = a + (-b)` | |

## Real Numbers (`import Mathlib.Data.Real.Basic`)

| Theorem | Type | Notes |
|---------|------|-------|
| `Real.sin_sq_add_cos_sq x` | `sin x ^ 2 + cos x ^ 2 = 1` | |
| `Real.sin_add x y` | `sin(x+y) = sin x cos y + cos x sin y` | |
| `Real.cos_add x y` | `cos(x+y) = cos x cos y - sin x sin y` | |
| `Real.sin_pi_div_two` | `sin(π/2) = 1` | |
| `Real.cos_pi` | `cos π = -1` | |
| `Real.exp_add x y` | `exp(x+y) = exp x * exp y` | |
| `Real.exp_zero` | `exp 0 = 1` | |
| `Real.log_mul` | `log(x*y) = log x + log y` | `x,y > 0` |
| `Real.sqrt_mul_self` | `√(x²) = |x|` | |
| `Real.sq_sqrt` | `(√x)² = x` | `x ≥ 0` |
| `Real.sqrt_sq_eq_abs` | `√(x²) = |x|` | |

## Complex Numbers (`import Mathlib.Data.Complex.Basic`)

| Theorem | Type | Notes |
|---------|------|-------|
| `Complex.I_sq` | `I^2 = -1` | |
| `Complex.exp_add_pi_mul_I` | `exp(π*I) = -1` | Euler's identity |
| `Complex.normSq_eq_conj_mul_self` | `normSq z = z * conj z` | |
| `Complex.abs` | `|z|` | modulus |
| `Complex.conj` | `bar z` | conjugate |

## Sets and Functions (`import Mathlib.Data.Set.Basic`)

| Theorem | Type | Notes |
|---------|------|-------|
| `Set.ext_iff` | `s = t ↔ ∀ x, x ∈ s ↔ x ∈ t` | |
| `Set.mem_insert_iff` | `x ∈ insert a s ↔ x = a ∨ x ∈ s` | |
| `Set.mem_union_iff` | `x ∈ s ∪ t ↔ x ∈ s ∨ x ∈ t` | |
| `Set.mem_inter_iff` | `x ∈ s ∩ t ↔ x ∈ s ∧ x ∈ t` | |
| `Set.subset_def` | `s ⊆ t ↔ ∀ x, x ∈ s → x ∈ t` | |
| `Set.image_eq_image` | | |
| `Set.range f` | `{ y | ∃ x, f x = y }` | |
| `Set.image_subset` | | |

## Order Theory (`import Mathlib.Order`)

| Theorem | Type | Notes |
|---------|------|-------|
| `le_of_lt h` | `a < b → a ≤ b` | |
| `lt_of_lt_of_le h1 h2` | `a < b ∧ b ≤ c → a < c` | |
| `lt_of_le_of_lt h1 h2` | `a ≤ b ∧ b < c → a < c` | |
| `le_trans h1 h2` | `a ≤ b ∧ b ≤ c → a ≤ c` | |
| `lt_trans h1 h2` | `a < b ∧ b < c → a < c` | |
| `le_refl a` | `a ≤ a` | |
| `le_antisymm h1 h2` | `a ≤ b ∧ b ≤ a → a = b` | |
| `by_cases h : P` | case split on any decidable prop | |
| `Nat.lt_of_lt_of_le` | ℕ version | |

## Tactic Automation (`import Mathlib.Tactic`)

| Tactic | Purpose | Example |
|--------|---------|---------|
| `ring` | polynomial identities | `ring` |
| `field_simp` | clear denominators | `field_simp [h]` |
| `linarith` | linear arithmetic | `linarith` |
| `nlinarith` | non-linear arithmetic | `nlinarith` |
| `omega` | ℕ/ℤ linear arithmetic | `omega` |
| `positivity` | prove positivity | `positivity` |
| `gcongr` | generalized congruence | `gcongr` |
| `grind` | SMT-style automation | `grind` |
| `aesop` | proof search | `aesop` |
| `norm_num` | normalize numeric expressions | `norm_num` |
| `exact?` | search for exact theorem | `exact?` |
| `apply?` | search for applicable lemma | `apply?` |
| `simp?` | show which lemmas simp uses | `simp?` |

## Common Search Patterns

When you don't know the exact theorem name, search by pattern:

```lean4
-- By keyword (local + mathlib):
lean_local_search("triangle inequality")

-- By type signature:
lean_loogle("|a + b| ≤ |a| + |b|")

-- By goal pattern (semantic):
lean_leanfinder("a + 0 = a")

-- Natural language:
lean_leansearch("commutativity of addition")
```
