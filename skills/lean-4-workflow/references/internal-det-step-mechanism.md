# internal_det_step — Self-contained 2×2 system (D3.2)

Discovered 2026-06-27 after multiple failed attempts with a single partner `(i,n)`.
The breakthrough was realizing that **three** bracketIdentity equations, not two,
form a closed 2×2 linear system with determinant 1 (char-independent).

## The mechanism

For adjacent source `(i,i+1)`, target `(2,i+1)`, `i ≥ 3`, `i+1 < n`:

```
  EQ1: bracketIdentity(1,2, i,n, 1,n)
       → coeff(1,2;1,i) + coeff(i,n;2,n) = 0

  EQ2: bracketIdentity(1,2, i,i+1, 1,i+1)
       → coeff(1,2;1,i) + coeff(i,i+1;2,i+1) = 0

  EQ3: coeffOf_cond(i,n;2,n) at split i+1
       → 2·coeff(i,n;2,n) = coeff(i,i+1;2,i+1)
```

Let `x = coeff(i,i+1;2,i+1)`, `y = coeff(i,n;2,n)`:

- EQ1 + EQ2: `x = y` (both = `-coeff(1,2;1,i)`)
- EQ3: `2y = x`
- Combine: `y = 2y → y = 0 → x = 0`

## Why partner (i,n) alone failed

The initial attempt used `bracketIdentity(i,i+1,i,n,u,i+1) → T4 → coeff(i,n;u,i)=0`,
then `coeffOf_cond(i,n;u,i)` at split `i+1`. This failed because:

- T4 gives coefficient at target `(u,i)` — target column = source row
- `coeffOf_cond` A-term guard is `"target column = source column"` → `i = n`
- When `i+1 < n`, `i ≠ n` → A-term always zero → no goal

The fix: use `bracketIdentity(1,2,i,n,1,n)` where `(1,2)` is the **first** source.
T1 fires: `1 < i ∧ n = n` → gives `coeff(1,2;1,i)` at target `(1,i)`. Then T3 gives
`coeff(i,n;2,n)` at target `(2,n)`. The target `(2,n)` IS the source column `n`,
making `coeffOf_cond`'s A-guard `2 < i+1 ∧ n = n` TRUE.

## Edge case: i+1 = n

When `i+1 = n`, the split `i+1 = n` equals the source column, so
`coeffOf_cond(i,n;2,n)` needs `i < i+1 < n` which fails when `i+1 = n`.
This case is unreachable from the dispatch — it's caught by BND-R first.
The lemma explicitly requires `hi1_lt_n : i+1 < n` as a hypothesis.

## Lean implementation

```lean4
lemma internal_det_step (D : HalfDerivation F n) (hn : 3 ≤ n) (hn5 : 5 ≤ n) (h3 : (3 : F) ≠ 0)
    (i : ℕ) (hi : 3 ≤ i) (hi1_n : i + 1 ≤ n) (hi1_lt_n : i + 1 < n) :
    coeffOf D i (i + 1) 2 (i + 1) = 0 := by
  -- EQ1: bracketIdentity(1,2, i,n, 1,n) = 0
  -- EQ2: bracketIdentity(1,2, i,i+1, 1,i+1) = 0
  -- EQ3: coeffOf_cond(i,n;2,n) at split i+1
  -- → x = y, 2y = x → y = 0 → x = 0
```

Key implementation details:

1. **Use `ring` over Field F, never `linarith`** — `linarith` fails on arbitrary fields.
   The closure `2y = y → y = 0` uses `((2:F)-1) * y = 0` via `ring`.

2. **`simpa` for bracketIdentity → coeffOf conversion** — `coeffOf_f` bridges `D.coeff`
   ↔ `coeffOf`. Use `rw [h1, h2'] at hb; simpa` (not `linarith`).

3. **`hi1_lt_n` is REQUIRED** — passed from the dispatch (BND-R catches `i+1=n`).

4. **`coeffOf_cond` B-guard**: `u = m ∧ j < v` = `2 = i+1 ∧ n < n`. The second
   conjunct `n < n` (NOT `n < i`) — this was the bug that cost multiple iterations.

## Axiom status

`lean_verify` returns `{propext, Quot.sound}` — no `sorryAx`, no `Classical.choice`.

## When to use this pattern

Use `internal_det_step` for any adjacent source `(i,i+1)` with target `(2,i+1)`
where `(2,i+1) ∉ I` and `i+1 < n`. This covers the A-fires and C-fires sorries in
`width_c_chain_zero`.

For the σ-mirror (`internal_det_step_a`, target `(1,n-1)`), see
`references/internal-det-step-a-sigma-mirror.md`.  It does NOT use the same
3-equation mechanism — the σ-mirror is a single `bracket_zero` with partner
`(n-1,n)` at target `(1,n)`, because T1 = goal fires at `1<n-1` and all other
terms vanish for interior sources.

## Comparison with adjacent-base-i

The `adjacent-base-i-implementation.md` recipe uses a commuting partner +
`width_stability_a/c` to kill a wide `∈ I` residual. `internal_det_step` is
different: it uses **three** equations where the third is `coeffOf_cond` (not a
frozen width lemma), forming a self-contained 2×2 system. This makes it
independent of `WidthFiltration` — it sits at the same DAG level.

Choose the right pattern by target:
- Target ∉ I, residual lands on wide ∈ I → use adjacent-base-i (width_stability)
- Target ∉ I, residual can be reached via coeffOf_cond → use internal_det_step
