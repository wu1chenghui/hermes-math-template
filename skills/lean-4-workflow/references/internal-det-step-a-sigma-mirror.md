# internal_det_step_a — σ-mirror via single bracket_zero (D3)

Discovered 2026-06-27.  The σ-mirror of `internal_det_step` does NOT use a
3-equation 2×2 system.  Instead, it uses a single `bracket_zero` with a
strategically chosen commuting partner.

## The mechanism

For adjacent source `(i,i+1)`, target `(1,n-1)`, `2 ≤ i ≤ n-3`:

```
  bracketIdentity(i,i+1; n-1,n; 1,n) = 0    (via bracket_zero, commuting sources)

  Expanded:
    T1 = coeff(i,i+1; 1, n-1)    ← GOAL (fires: 1 < n-1 ∧ n = n)
    T2 = 0                        (1 ≠ n-1)
    T3 = 0                        (i ≠ 1, since i ≥ 2)
    T4 = 0                        (i+1 ≠ n, since i ≤ n-3)
```

The partner is `(n-1,n)` — the σ-image of `(1,2)` under the anti-automorphism
`E_ab ↦ E_{n+1-b, n+1-a}`.  But crucially, the **proof structure changes**:
the original uses 3 equations (2 bracketIdentity + 1 coeffOf_cond), while
the σ-mirror uses just 1 bracket_zero.

## Why it's simpler

The original `internal_det_step` needs 3 equations because:
- bracketIdentity(1,2, i,n, 1,n) relates coeff(i,n;2,n) to coeff(1,2;1,i)
- bracketIdentity(1,2, i,i+1, 1,i+1) relates coeff(i,i+1;2,i+1) to coeff(1,2;1,i)
- coeffOf_cond(i,n;2,n) at split i+1 gives 2·coeff(i,n;2,n) = coeff(i,i+1;2,i+1)
- Together: x=y, 2y=x → y=0 → x=0

In the σ-mirror, bracketIdentity(i,i+1; n-1,n; 1,n) has **only T1 firing**
because the target `(1,n)` has column `n`, so `v = d = n` kills T2, and
`u = 1 ≠ i` (for i ≥ 2) kills T3, and `n ≠ i+1` (for i ≤ n-3) kills T4.
The single surviving term IS the goal.  No coupling, no linear system.

## Boundary cases (sorried)

| i | Source | Issue | Handled by |
|---|--------|-------|------------|
| 1 | (1,2) | T3 fires → coupling with coeff(n-1,n;2,n) | width_a chain (k=1 branch) |
| n-2 | (n-2,n-1) | [E_{n-2,n-1}, E_{n-1,n}] = E_{n-2,n} ≠ 0 | width_a chain (k=n-2) |
| n-1 | (n-1,n) | T4 fires → T1=T4 cancellation → 0=0 useless | width_a chain (k=n-1) |

These are analogous to how `internal_det_step` requires `i ≥ 3` and `i+1 < n`
(boundary cases handled by the width_c chain's induction dispatch).

## Lean implementation

```lean4
lemma internal_det_step_a (D : HalfDerivation F n) (hn : 3 ≤ n) (hn5 : 5 ≤ n) (h3 : (3 : F) ≠ 0)
    (i : ℕ) (hi1_n : i + 1 ≤ n) (ih : ...) :
    coeffOf D i (i + 1) 1 (n - 1) = 0 := by
  by_cases hi0 : i = 0
  · subst hi0; unfold coeffOf; simp
  have hi : 1 ≤ i := by omega
  by_cases hi_eq_1 : i = 1
  · subst hi_eq_1; sorry  -- boundary
  by_cases hi1_eq_n1 : i + 1 = n - 1
  · sorry  -- i = n-2
  by_cases hi1_eq_n : i + 1 = n
  · sorry  -- i = n-1
  · -- Clean interior: 2 ≤ i ≤ n-3
    have h_n3 : 1 < n - 1 := by omega
    have hb : bracketIdentity D.coeff i (i+1) (n-1) n 1 n = 0 :=
      D.bracket_zero i (i+1) (n-1) n 1 n
        hi (by omega) hi1_n
        (by omega) (by omega) (le_refl n)
        (by omega) (by omega) (le_refl n)
        (by omega) (by omega)
    rw [bracketIdentity_eq_expanded] at hb
    have c1 : (1 : ℕ) < n - 1 ∧ (n : ℕ) = n := ⟨h_n3, rfl⟩
    have c2 : ¬((1 : ℕ) = n - 1 ∧ (n : ℕ) < n) := by omega
    have c3 : ¬((1 : ℕ) = i ∧ (i+1 : ℕ) < n) := by omega
    have c4 : ¬((1 : ℕ) < i ∧ (n : ℕ) = i+1) := by omega
    rw [if_pos c1, if_neg c2, if_neg c3, if_neg c4] at hb
    rw [coeffOf_f D i (i+1) 1 (n-1) hi (by omega) hi1_n (by omega) h_n3 (by omega)]
    simpa using hb
```

## Key lesson: σ-mirrors don't preserve proof structure

The anti-automorphism `σ: E_ab ↦ E_{n+1-b, n+1-a}` maps:
- `(1,2)` ↔ `(n-1,n)`  (partner)
- `(2,n)` ↔ `(1,n-1)`  (I targets)
- `(i,i+1)` ↔ `(n-i, n-i+1)` (sources)

But the **proof mechanism changes**: the original uses a 2×2 linear system
because the target `(2,i+1)` is absorbed differently in bracketIdentity's
T1–T4 guards.  The σ-mirror's target `(1,n-1)` happens to simplify to a
single-term identity under the right bracket partner.  Always re-derive
the expanded bracketIdentity terms for the σ-mirror — never assume the same
proof structure.