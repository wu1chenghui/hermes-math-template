# Chain induction via Nat.decreasingInduction

Use when: proving a lemma P(ℓ) for all ℓ ≤ n by descending induction on ℓ,
where the descent step splits a source at ℓ+1 and children are either
(a) in the same cluster with larger ℓ (→ induction hypothesis) or
(b) terminal/export cases (→ other lemmas).

## The Nat.decreasingInduction API (mathlib4)

```lean
Nat.decreasingInduction {n : ℕ} {motive : (m : ℕ) → m ≤ n → Sort u}
  (of_succ : (k : ℕ) → (h : k < n) → motive (k + 1) h → motive k (Nat.le_of_lt h))
  (self : motive n (le_refl n))
  {m : ℕ} (mn : m ≤ n) : motive m mn
```

Key gotchas:
- **Dependent motive**: `motive` takes TWO args: `(ℓ : ℕ)` AND `(ℓ ≤ n)`. Not just ℕ.
- **`h : k < n` is `k.succ ≤ n`**: In mathlib4, `a < b` = `a.succ ≤ b`. So `h : k < n`
  serves as the proof that `k+1 ≤ n` for `motive (k+1) h`. This is NOT `Nat.le_of_lt h`.
- **Return**: `motive k (Nat.le_of_lt h)` — the second argument is `Nat.le_of_lt h : k ≤ n`.
- **`let`-bound predicates**: Use `(motive := P)` explicitly; the elaborator can't infer
  a `let`-bound lambda as the motive.
- **`set`-bound variables**: `set m := k + 1 with hm` creates a binder that `rw` can't
  match. Use `dsimp [m]` to unfold it.

## The B1→B2→B3 decomposition for chain induction

When proving a chain induction lemma (e.g. `width_c_chain_zero`) where the
descent step involves `coeffOf_cond_of` with multiple child types:

**B1 — Freeze the induction skeleton** (signature, measure, dispatch structure).
  Leave all child analysis as `sorry`. Goal: induction shape compiles, statement
  won't change.

**B2 — Close all recursive edges** that can be killed by the induction hypothesis
  or existing trivial cases (char≠2, all-children-zero, degenerate indices).
  Leave terminal / boundary-coupling / cluster-export cases as `sorry`.

**B3 — Close terminal cases** (boundary coupling, cluster exports, width-2).
  Wire to existing lemmas like `boundary_coupling` (GATE-A).

## Chain induction case analysis pattern

When the descent step `coeffOf_cond_of` produces 4 children (A,B,C,D), each
guarded by an `if` on index conditions:

1. **Eliminate D** first (always false for the given index ordering).
2. **Use `simp` at hcond** with each child's condition, rather than manually
   constructing `hXchild` equalities and `rw`-ing. `simp [cond] at hcond` is
   more robust when `set`/`subst` have altered binder shapes.
3. **Case-split on A, B, C** with `by_cases`. Handle each branch.
4. **All-zero branch**: `simp` all conditions to 0, then `simpa [sub_eq_add_neg, ...]`
   using hcond — prefer over `linear_combination` which fails when hcond is
   already simplified to `2*x = 0`.

## coeffOf returns 0 for invalid indices

`HalfDerivation.coeffOf D i j u v` (from `HalfProjection.lean`) uses internal
`if` guards:
```
if hi : 1 ≤ i then if hij : i < j then if hjn : j ≤ n then
  if hu : 1 ≤ u then if huv : u < v then if hvn : v ≤ n then D.coeff i j u v else 0
  else 0 else 0 else 0 else 0
```
So `coeffOf D 0 j' 2 n = 0` is trivially provable by `unfold coeffOf; simp`.

## `subst` pitfalls in dependent contexts

- `subst` can clear section variables (`n`) from context, causing "unknown identifier"
  errors downstream. Prefer `rw [h]` over `subst h` inside nested `have` blocks.
- `subst` on a `set`-bound variable fails with "invalid equality proof". Use
  `dsimp` or `unfold` + `rw` instead.
