# Induction + `generalizing` Pitfall (and Universal-Quantifier Trap)

Two related Lean 4 patterns that caused >10 compile-error iterations in the
`centered_image_in_I` proof (Centering.lean, 2026-06-29).

---

## Pitfall 1: `induction' … generalizing i j` + `revert` interaction

### What happened

```lean
-- Context has: hi, hij, hjn, h_off_diag, h_ge2, hcoeffOf_ne
revert hi hij hjn h_off_diag h_ge2 hcoeffOf_ne
induction' hlen : n-u using Nat.strong_induction_on with d ih generalizing i j
intro hi hij hjn h_off_diag h_ge2 hne
```

**Expected**: `intro` binds in the same order as `revert`.
**Actual**: `generalizing i j` moved ADDITIONAL context hypotheses (that mention
`i` or `j` but were NOT reverted — e.g. `hne✝`, `h_notI`, `hcoeff`, `h_diag`,
`hcoeff_n`, `h_adj`) into the goal BEFORE the reverted ones. The `intro` order
was completely scrambled, and the induction hypothesis `ih` had 15+ arguments
instead of the expected 7.

### Why

`generalizing` in `induction'` generalizes ALL hypotheses in context that
mention the generalized variables, not just the reverted ones. `revert` makes
them explicit arguments; `generalizing` makes them implicit (quantified by `∀`).
When BOTH are used together, the generalized-from-context arguments appear FIRST
in the `∀`-quantified type, and the reverted arguments appear AFTER.

### The fix (for this case)

**Option A** (preferred when everything is already reverted): drop `generalizing`.
```lean
revert hi hij hjn h_off_diag h_ge2 hcoeffOf_ne
induction' hlen : n-u using Nat.strong_induction_on with d ih   -- NO generalizing
intro hi hij hjn h_off_diag h_ge2 hne
```
The `ih` then takes `(measure)(proof-of-small) (i j) (all-7-intro-args)`.

**Option B** (when using `generalizing` as in `offdiag_zero_v_lt_n`): revert
ONLY ONE hypothesis, and let `generalizing` move everything else.
```lean
revert hvn_lt  -- only ONE thing reverted
induction' hlen : v-u using Nat.strong_induction_on with d ih generalizing i j u v
intro hvn_lt
-- ih now has ALL original hypotheses as arguments, in their context order
```

**Rule of thumb**: either `revert` everything AND drop `generalizing`, or
`revert` one thing AND use `generalizing`. Mixing full `revert` with
`generalizing` creates an opaque argument soup.

---

## Pitfall 2: Universal `∀ a b c d e f, Phi coeff … = 0` is a FALSE proposition

### The trap

The adjacent v=n helper lemmas (`adjacent_rowcol_zero`, `adjacent_below_zero`,
etc.) originally had signatures like:
```lean
(hPhi : ∀ i j c d u v, Phi coeff i j c d u v = 0) : coeff i (i+1) u n = 0
```

This looks innocuous — "Phi is zero for all indices." But `Phi` is defined as
`2*bracketSource - bracketIdentity`, and `bracketSource`/`bracketIdentity` do
NOT check source validity (`i < j`, `c < d`). For invalid indices (e.g. `i ≥ j`),
`Phi` is NOT guaranteed to be zero — `half_leibniz` only holds for valid indices.

**The universal quantifier `∀ a b c d e f` is a type-theoretic LIE.**

It cannot be proved from `halfDerivation_phi_zero` because that lemma requires
9 validity hypotheses. The only way to "prove" it would be `by_cases` on
validity + `unfold Phi bracketSource bracketIdentity; split_ifs` for the
invalid cases — a 9-deep case explosion.

### The fix

Change each helper's signature from a universal `hPhi` to the SPECIFIC
`Phi = 0` hypothesis it actually needs:

```lean
-- BEFORE (wrong):
theorem adjacent_below_zero (hPhi : ∀ i j c d u v, Phi coeff i j c d u v = 0) …

-- AFTER (correct):
theorem adjacent_below_zero (hP : Phi coeff 1 u i (i+1) 1 n = 0) …
```

The specific indices (`1, u, i, i+1, 1, n`) are exactly the ones used in the
helper's proof body. They always satisfy all validity conditions (enforced by
the helper's own hypotheses like `hip1_lt_n`, `hu3`, etc.).

At each call site in `centered_image_in_I`, construct the specific `hP` using:
```lean
have hP := halfDerivation_phi_zero D' 1 u i (i+1) 1 n
  (by omega) (by omega) (by omega) (by omega) (by omega) (by omega)
  (by omega) (by omega) (by omega)
```

This is clean, local, and type-correct. It also documents exactly WHICH
half-Leibniz equation is being used at each dispatch point (improving
auditability).

### General principle

**When a helper lemma needs `Phi = 0` at specific indices, make those indices
explicit in the hypothesis. Never quantify universally over all indices unless
the statement is actually true for all indices (with a validity guard).
`Phi` without validity guards is NOT universally zero.**

---

## `subst` destroys section variable `n` — reinforced

This is already in the skill's pitfall section, but the v=n case is a
particularly dangerous instance. When `n` is a section variable (from
`variable (n : ℕ)`), `subst hvn_eq` where `hvn_eq : v = n` will clear `n`
from the context entirely, causing "Unknown identifier `n`" errors at every
downstream use of `n` (including in lemma calls like `(n := n)`).

**Always prefer `rw [hvn_eq]` over `subst hvn_eq` when `n` is a section
variable.**
