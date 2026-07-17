# ONLY Lemma Proof Pattern (NnDerivation architecture)

Proven June 2026 during the HalfDerivation → NnDerivation refactoring.

## Context

When a bracket identity `[D(E_i,i+1), E_cd] + [E_i,i+1, D(E_cd)] = 0` is
projected to basis element E_uv, we get:

```
bracketIdentity = A − B + C − D = 0
```

where each of A,B,C,D is an `if cond then coeff else 0` expression.

An "ONLY" lemma (only_A, only_B, only_C, only_D) states: when exactly ONE of
the four conditions is active, that single active coefficient must be zero.

## The Pattern

For each ONLY lemma, the proof follows this template:

```lean4
theorem only_X (D : NnDerivation F n) ... :
    coeffOf D source_target = 0 := by
  rcases hX_cond with ⟨condition_hyp1, condition_hyp2⟩
  have hij : i < i+1 := by omega
  -- Step 1: Get the bracket identity
  have h_bracket := bracket_zero_coeff D i (i+1) c d u v hi hij hip1n hc hcd hdn hu huv hvn
    hp1_ne_c (Ne.symm hd_ne_i)
  -- Step 2: Compute each of the four if-expressions independently
  have hA_val_or_zero : (if u < c ∧ v = d then D.coeff i (i+1) u c else 0) = <0 or coeff> := by
    simp [<condition or its negation>]
  have hB_val_or_zero : ... := by simp [...]
  have hC_val_or_zero : ... := by simp [...]
  have hD_val_or_zero : ... := by simp [...]
  -- Step 3: Combine into bracketIdentity
  unfold bracketIdentity bracketLeft bracketRight at h_bracket
  simp [hA_val_or_zero, hB_val_or_zero, hC_val_or_zero, hD_val_or_zero] at h_bracket
  -- Step 4: h_bracket is now <active_coeff> = 0 (or -(active_coeff) = 0)
  -- Convert D.coeff to coeffOf and close
  have hcoeff : coeffOf D ... = D.coeff ... := coeffOf_f D ... <validity proofs>
  rw [hcoeff]
  exact h_bracket  -- or `neg_eq_zero.mp h_bracket`
```

## Key Insights

1. **Each `have` block uses `simp` with exactly the conditions that determine that term.**
   `simp [hA_false]` works because it's a SINGLE if-expression, not nested.
   This avoids the `simp` + `¬(P∧Q)` problem entirely.

2. **Never use `split_ifs` with 4 conditions.** It creates 16 cases, most of
   which are contradictory and must be dispatched with `omega`.

3. **Never try to `simp [hA_false, hB_true, ...]` the whole expression at once.**
   `simp` can't use `¬(P∧Q)` to rewrite nested if-expressions. Decompose first.

4. **Use `Ne.symm hd_ne_i` when the bracket_zero_coeff lemma expects `i ≠ d`
   but you have `d ≠ i`.** The parameter is named `hi_ne_l : i ≠ l`.

## Dependencies

- `bracket_zero_coeff` (from Coefficient.lean) — the sole source of the bracket identity
- `coeffOf_f` (from Projection.lean) — converts D.coeff to coeffOf
- `omega` — for arithmetic contradictions
- `neg_eq_zero.mp` — for `-X = 0 → X = 0` (linarith doesn't work on Field F)

## The Four Lemmas

| Lemma | Active term | Inactive terms enforced by | Result |
|-------|------------|---------------------------|--------|
| only_A | A: coeff(i,i+1;u,c) | ¬(u=c), ¬(u=i∧i+1<v), ¬(u<i∧v=i+1) | coeff(i,i+1;u,c) = 0 |
| only_B | B: coeff(i,i+1;d,v) | ¬(u<c), ¬(u=i∧i+1<v), ¬(u<i∧v=i+1) | coeff(i,i+1;d,v) = 0 |
| only_C | C: coeff(c,d;i+1,v) | ¬(u<c∧v=d), ¬(u=c∧d<v), ¬(u<i) | coeff(c,d;i+1,v) = 0 |
| only_D | D: coeff(c,d;u,i) | ¬(u<c∧v=d), ¬(u=c∧d<v), ¬(u=i) | coeff(c,d;u,i) = 0 |
