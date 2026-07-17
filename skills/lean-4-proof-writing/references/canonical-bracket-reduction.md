# Canonical 4-Term bracketIdentity Reduction at a Fixed Output

> Technique from the dim=n+5 formalization (HalfDerivation classification, 2026-06-23).
> Source: `Spanning.lean` in the project at `/opt/lean-home/lean-projects/e/E/Classification/`.

When `bracketIdentity coeff i j c d u v = 0` (obtained from `Φ=0` + commuting or
`kerΦ` membership), expanding at a specific output `(u,v)` yields four indicator
terms.  The canonical form reduces the generic `bracketIdentity_eq_expanded`
guards to a clean 4-term sum by applying `I_filtered` pruning **before**
expanding into if-cascade.

## When to use

- You have a coefficient-level identity `bracketIdentity coeff ... u v = 0` and
  need to extract a scalar relation among specific channel entries.
- You are working with `Φ: Hom(N_n, I) → Hom(N_n∧N_n, I)` (half-Leibniz operator)
  and need to project onto adjacent channel coordinates `(a_p, b_p, c_p)`.

## The template

### Step 1 — Semantic pruning (U0)

Apply `I_filtered` to kill all output channels except the single survivor `(1,n)`.
This is done by two lemmas, one per non-`(1,n)` I-channel:

```lean
lemma bracketIdentity_a_output_zero (hn : 3 ≤ n) {coeff : …} (hI : I_filtered coeff hn)
    (i j c d : ℕ) (hij : i < j) (hcd : c < d) :
    bracketIdentity coeff i j c d 1 (n - 1) = 0 := by
  rw [bracketIdentity_eq_expanded]
  have z1 : (if (1 : ℕ) < c ∧ n - 1 = d then coeff i j 1 c else 0) = 0 := by
    split_ifs with h
    · obtain ⟨h1, h2⟩ := h
      refine coeff_zero_of_not_I hn hI i j 1 c ?_
      simp only [I_target_set, Set.mem_insert_iff, Set.mem_singleton_iff, Prod.mk.injEq]; omega
    · rfl
  …  -- same for z2,z3,z4; then rw [z1,z2,z3,z4]; ring
```

The companion `bracketIdentity_c_output_zero` for output `(2,n)` is identical
except the guard numbers change.  Combined they give `bracketIdentity_reduction_to_1n`.

### Step 2 — Canonical 4-term expansion (U2 — THE gate)

This is the single expansion point.  **Do not expand before this lemma.**

```lean
lemma bracketIdentity_at_1n_canonical (hn : 3 ≤ n) {coeff : …} (hI : I_filtered coeff hn)
    (i j c d : ℕ) (hij : i < j) (hcd : c < d) :
    bracketIdentity coeff i j c d 1 n =
      (if c = n - 1 ∧ d = n then coeff i j 1 (n - 1) else 0)   -- T1
    - (if c = 1     ∧ d = 2 then coeff i j 2 n       else 0)   -- T2
    + (if i = 1     ∧ j = 2 then coeff c d 2 n       else 0)   -- T3
    - (if i = n - 1 ∧ j = n then coeff c d 1 (n - 1) else 0)   -- T4
```

Each of the four terms T1–T4 is a named `have hTk` block.  Each block
uses `by_cases hg` (source guard from `bracketIdentity_eq_expanded`) and
`by_cases ht` (target guard — the clean I-filtered guard).  Four subcases
per block:

| hg | ht | handler |
|----|----|---------|
| true | true | `subst` + `simp` + `omega` (both guards fire) |
| true | false | `coeff_zero_of_not_I` kills the inner read → `simp [h_noI, hg, ht]` |
| false | true | impossible (hg contradicts ht) → `exfalso; omega` |
| false | false | both sides 0 → `simp [hg, ht]` |

### Step 3 — Apply commuting (U1)

`bracketSource_commuting_eq_zero` is a standalone 1-liner: for commuting pair
(`j ≠ c`, `i ≠ d`), `bracketSource coeff i j c d u v = 0` for every target.
Proof: `unfold bracketSource; rw [if_neg hjc, if_neg hid]; ring`.

### Step 4 — Chain into scalar relation

From `Φ coeff i j c d 1 n = 0`:

1. `bracketSource_commuting_eq_zero` → `bracketSource = 0`
2. `Phi = 2·bracketSource − bracketIdentity = 0` + bracketSource=0 → `-bracketIdentity = 0`
   → `bracketIdentity = 0` via `neg_eq_zero.mp` (NOT `linarith`).
3. `bracketIdentity_at_1n_canonical` rewrites `bracketIdentity` into 4-term sum.
4. Named `hT1`–`hT4` rewrites simplify the sum to a scalar equation.

## Pitfalls

### `simp` with `¬(p ∧ q)` does not simplify `if p ∧ q then …`

If you have `h : ¬(p ∧ q)`, writing `simp [h]` to simplify `if p ∧ q then a else b`
**does not work** because `simp` does not use `¬(p ∧ q)` to destruct the
conjunction inside the `if` guard.  Instead:

- **For the full conjunction guard**: extract conjuncts first, then use those as
  simp arguments (e.g., `simp [hp, hq]` where `hp : ¬p` or `hq : ¬q`).
- **For terms T1–T4 where each guard is a single equality**: `simp [show … ≠ … from by omega]`
  works because `simp` can apply `if_neg` to `¬(a = b)`.

Failing that, use an explicit `calc` with `rw`:

```lean
  have hT2 : (if (n-1 : ℕ) = 1 ∧ n = 2 then coeff … else 0) = 0 := by
    simp [show (n-1 : ℕ) ≠ 1 from by omega, show n ≠ 2 from by omega]
  rw [hT2] at h_bi_eq_0
```

### `rw` not `simp` for named hT* rewrites

When you have `hTk : old_if = new_term`, applying it to a hypothesis `h`:

- `rw [hTk] at h` — works, replaces `old_if` by `new_term` in `h`.
- `simp [hTk] at h` — **does not work** because `simp` rewrites but may
  not match the exact subterm structure (parentheses differ).

### `linarith` fails on `-x = 0`

`linarith` works over commutative semirings but does not handle the field
subtraction/negation pattern `-x = 0 → x = 0`.  Use:

```lean
    exact neg_eq_zero.mp hPhi_val   -- given hPhi_val : -bracketIdentity … = 0
```

### `calc` cannot mix `<` and `=` chains

To prove `i < n` from `hij : i < j` and `hn_eq : n = j`:

```lean
    have hi_lt_n : i < n := by
      rw [← hn_eq]; exact hij
    -- OR
    exact lt_of_lt_of_eq hij hn_eq.symm
    -- OR simply
    omega
```

`calc i < j := hij; _ = n := hn_eq.symm` fails because calc expects all
steps to use the same relation.

## Reference

- `sparse-coeff-bracket-identity.md` — coordinate-level `split_ifs` explosion
  avoidance (complements the 4-term canonical pattern).
- `field-f-pitfalls.md` — general field arithmetic pitfalls (omega, ring, linarith).
