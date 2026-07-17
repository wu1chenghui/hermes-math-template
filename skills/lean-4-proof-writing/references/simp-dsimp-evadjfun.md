# simp vs dsimp for evAdjFun / coeff-identity lemmas

When proving a goal of the form

```lean
evAdjFun (coeff_X ...) = (a_ch, b_ch, c_ch)
```

the natural first step is `simp [evAdjFun, coeff_X]` to unfold the definitions.
This **often fails** because:

## The problem

`simp` with `coeff_X` (which is a nested `if` expression) **runs `split_ifs`
internally** as part of its simplification loop. The result is one of two failure
modes:

1. **"No goals to be solved"** — `simp` already closed the entire goal (rare, but
   happens for components where all guards are identically false, e.g. `coeff_B`
   and `coeff_E`). The subsequent `split_ifs` or `omega` has nothing to do.

2. **"split_ifs: no if-then-else conditions to split"** — `simp` partially
   reduced the `if`, leaving a residue goal like `p.val+1 = i → ¬(n-1 = n)`.
   The `if` has been consumed; `split_ifs` finds no target to split.

Both failures are *silent* in the sense that `simp` + `split_ifs` looks like it
should work — the problem is that `simp` pre-consumed the `if`.

## The fix

Use **`dsimp`** instead of `simp` to unfold definitions. `dsimp` only uses
definitional equality (δ-reduction); it does NOT run `split_ifs` or any
propositional simplification.

```lean
-- ❌ Undefined behaviour — may close, may leave phantom goals
ext p; simp [evAdjFun, coeff_X]; split_ifs <;> first | rfl | exfalso; omega

-- ✅ Predictable — dsimp unfolds, split_ifs creates branches, omega closes them
ext p; dsimp [evAdjFun, coeff_X]; split_ifs <;> first | rfl | exfalso; omega
```

## When to use which

| Situation | Use |
|-----------|-----|
| Goal is an equality of `if`-expressions, coeff has nested ifs | `dsimp; split_ifs` |
| Goal is `coeff_X ... = 0` and guard is trivially false | `simp [coeff_X]` |
| Goal is `coeff_X ... = value` with specific indices | `simp [coeff_X]` if the indices force the guard |
| Bracket identity (`bracketSource = ...`) proof | `simp only [..]` (per sparse-coeff reference) |

## Concrete example (Spaaning.lean P2, 2026-06-23)

`evAdjFun_coeff_D` (coeff_D has a double-if with guards `si=n-2 ∧ sj=n-1` and
`si=n-2 ∧ sj=n`). For the a-channel component (target `(1,n-1)`):

After `simp [evAdjFun, coeff_D]` the elaborator left `p.val+1 = n-1 → ¬(n=n-1)`
— the `if` was already split. Switching to `dsimp [evAdjFun, coeff_D]` gave the
full `if` expression, and `split_ifs` generated the expected 3 branches:

1. `h₁`: first guard true → value=2, RHS=2 → `rfl`
2. `h₂`: first guard false, second guard true → `tv=n-1` vs `tv=n` contradiction → `exfalso; omega`
3. Neither guard true → value=0, RHS=0 → `rfl`

All 15 components across 5 lemmas (A,C,F,D,E) used the identical pattern and
compiled in one shot.

## Pitfall: `subst` then `simp` leaves Fin inequality goals

After `dsimp; split_ifs`, some branches use `subst hp2` to substitute
`p = ⟨n-2, ...⟩` into the goal. The resulting `simp` needs to resolve
`Fin` equalities like `⟨n-2, ...⟩ = ⟨0, ...⟩` in `if` conditions. `simp`
cannot close these — even `omega` can't handle `Fin` values (it works on
`ℕ`/`ℤ`/`ℚ`, not `Fin`).

**Fix**: provide explicit inequality helpers using `congrArg Fin.val`:

```lean4
· have hn20 : (⟨n-2, by omega⟩ : Fin (n-1)) ≠ (⟨0, by omega⟩ : Fin (n-1)) := by
    intro h; have := congrArg Fin.val h; omega
  subst hp2; simp [hn20]
```

The `congrArg Fin.val h` extracts the `ℕ` equality from `Fin` equality, giving
`n-2 = 0` (impossible for `n ≥ 4`). Wrapping it as a `≠` hypothesis feeds it
to `simp` which uses `h : a ≠ b` to close `a = b` goals in `if` conditions.

This pattern was needed at 5 positions across P3a (a-channel: n-3≠0, n-2≠0,
n-2≠n-3; c-channel: 1≠0, n-2≠0, n-2≠1). All use the identical 3-line template.

## Pitfall: `p.property` does not exist on `Fin n`

Accessing the bound proof on a `Fin n` element uses `.2` (the second field of
the subtype `{ val : ℕ // val < n }`), NOT `.property`:

```lean4
-- ❌ Invalid field 'property'
have : p.val < n-1 := p.property

-- ✅ Correct
have : p.val < n-1 := p.2
```

## Related

- `../sparse-coeff-bracket-identity.md` — taming `split_ifs` explosion in
  bracket-identity HALF-LEIBNIZ proofs (use `simp only`, not `simp`).
- This reference covers ADJDATA/evAdjFun proofs (use `dsimp`, not `simp`).

## P3b-style: evAdjFun of a SUM of coefficient functions

When computing `evAdjFun` of a linear combination of generators (the P3b task),
the obvious approach — expand `evAdjFun S_val`, `ext p` on each channel, and
`simp` with coeff definitions — hits a combination of problems:

1. `dsimp [evAdjFun, coeff_A, coeff_B, ...]` creates a massive goal with nested
   `if` inside `Finset.sum` → `simp` can't reduce it.
2. `calc` with `Finset.sum_congr` works for the A-part (Dirac delta sum) but
   the `ext p` creates an environment where the `calc` LHS doesn't match the
   goal after partial `simp`.
3. `LinearMap.map_sum` is NOT available with `import Mathlib.Tactic` only
   (it requires `Mathlib.LinearAlgebra.Basic`).

**Preferred approach (for next session):** Use the P2 `evAdjFun_coeff_X` lemmas
(already proven in Spanning.lean) and exploit that `evAdjFun` is definitionally
pointwise — `evAdjFun (f + g) = evAdjFun f + evAdjFun g` is `rfl`:

```lean
-- Decompose via rfl (definitional) rather than a lemma
have h_sum : evAdjFun (∑ p, bs p * coeff_A ...) = ∑ p, bs p * evAdjFun (coeff_A ...) := rfl
-- Now use evAdjFun_coeff_A lemma for each term
simp_rw [evAdjFun_coeff_A ...]
-- Then Finset.sum_congr + ext p
```

This avoids expanding coeff definitions entirely and keeps the proof at the
P2-level coordinate API. The only remaining work is the `Finset.sum` identity
for the Dirac delta (b-channel A-part).

If `map_sum` is needed, ensure `import Mathlib.LinearAlgebra.Basic` is in the
project's `.olean` closure before using it.
