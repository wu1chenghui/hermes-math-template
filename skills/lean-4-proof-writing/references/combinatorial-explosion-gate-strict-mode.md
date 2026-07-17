# Handling combinatorial explosion gates — strict mode case-split design

When a Lean proof reaches a **combinatorial explosion gate** (a single lemma
whose natural case analysis has 8+ `if` branches), throwing `split_ifs`,
`simp`, or `omega` at the whole goal leads to elaborator-dependent order
sensitivity — different elaboration paths produce different goal shapes, and
a proof that works by trial-and-error in one order may break after a seemingly
unrelated edit.

This document captures a **referee-enforced strict mode** for these gates. It is
a design pattern, not a tactic reference.

---

## Core principle: semantic pruning before expansion

**Never** expand an expression into its `if`-structured form before eliminating
the cases that are structurally impossible using domain invariants
(`I_filtered`, commutativity, validity guards, range constraints).

Wrong (forces the entire case tree through the elaborator at once):
```lean
lemma messy (hI : I_filtered coeff hn) ... : ... := by
  rw [bracketIdentity_eq_expanded]
  split_ifs <;> simp [hI]  -- BAD: 16 branches, order-dependent
```

---

## The 5-step protocol

### Step 1 — Semantic pruning (separate lemmas)

Write separate lemmas that prove the domain-invariant-based vanishings.
These do NOT expand the target expression — they work on the definition level.

Examples:
- `coeff_zero_of_not_I`: `I_filtered` ⟹ any coeff at a non-`I` target is 0.
- `bracketIdentity_a_output_zero`: bracketIdentity at output `(1,n-1)` vanishes
  (all inner-coeff reads are I-filtered-dead).
- `bracketSource_commuting_eq_zero`: for commuting source pairs `(i,j),(c,d)`,
  the source term is identically 0 (no `if` analysis needed — definitional).

Each lemma compiles independently. If one fails, the rollback unit is that single
lemma, not the whole gate.

### Step 2 — Single expansion point

One and only one `rw [expanded_form]` in the whole proof. No `simp` or `unfold`
that would expose `if` structure before this line. The expanded form is stored
as a lemma (e.g. `bracketIdentity_eq_expanded`), not inlined.

The lemma's RHS should express the expansion in the form closest to the
**canonical structure** of the problem, not merely the definitional unfolding.

### Step 3 — Named term bindings

Each indicator term of the expanded form is bound to a named `hTk` block:

```lean
rw [bracketIdentity_eq_expanded]    -- single expansion point

have hT1 : term_source_1 = term_target_1 := ...
have hT2 : term_source_2 = term_target_2 := ...
...
rw [hT1, hT2, hT3, hT4]
```

**Forbidden**: `match` on the expanded form, `by_cases`/`split_ifs` at the
full-goal level, inline `if`-cascades that span multiple terms.

### Step 4 — Fixed-order case analysis per term

Each `hTk` is proved independently. The template (both guards are conjunctions):

```lean
have hTk : source_term = target_term := by
  by_cases hg : source_guard    -- the inner-if guard from the expansion
  · obtain ⟨h_conjuncts...⟩ := hg
    by_cases ht : target_guard  -- the cleaned, post-pruning guard
    · -- both true: substitute and simplify
      subst ...; simp; omega
    · -- source true, target false: value forced to 0 by I_filtered / other invariant
      have h_zero : inner_coeff = 0 :=
        coeff_zero_of_not_I hn hI ... (by
          simp [I_target_set, ...] ; ...)
      -- Use `calc` with explicit LHS/RHS helpers (safer than `simp` on the whole goal)
      have hLHS : source_term = 0 := by simp [h_zero]
      have hRHS : target_term = 0 := by simp [ht]
      calc
        source_term = 0 := hLHS
        _ = target_term := by symm; exact hRHS
  · by_cases ht : target_guard
    · -- source false, target true: impossible (index constraints contradict)
      exfalso; apply hg
      refine ⟨by ..., ...⟩    -- use the equality from ht to rewrite, then omega
    · -- both false: 0 = 0
      simp [hg, ht]
```

#### Signals for Step 4 sub-branches

| branch | condition | proof pattern |
|--------|-----------|---------------|
| hg ∧ ht | both true | `subst ...; simp; omega` |
| hg ∧ ¬ht | source true, target false | `h_zero` via invariant → `calc` with `hLHS`/`hRHS` |
| ¬hg ∧ ht | source false, target true | `exfalso; apply hg; refine ...` |
| ¬hg ∧ ¬ht | both false | `simp [hg, ht]` |

#### Warning: `simp` does NOT destruct conjunction guards

`simp [ht]` where `ht : ¬(A ∧ B)` has NO effect on `if (A ∧ B) then ... else ...`.
`simp` only rewrites an `if` when the hypothesis matches the condition exactly
(syntactically), and `A ∧ B` is not syntactically matched by `¬(A ∧ B)`.

If you get an `unusedSimpArgs` warning for a hypothesis of type `¬(A ∧ B)`:
the hypothesis is being ignored. Use `calc` with explicit `hRHS` helper or
`split_ifs` on the specific term, not `simp [h]`.

#### Proving `I_target_set` membership refutation

Use `rcases` with explicit binders (NOT `rfl` patterns, which cause `subst`
errors from nested conjunction patterns).

```lean
simp only [I_target_set, Set.mem_insert_iff, Set.mem_singleton_iff, Prod.mk.injEq] at mem
rcases mem with (⟨h1, h'⟩ | ⟨h2, h'⟩ | ⟨h12, h'⟩)
· exact hi_n1 h'       -- h' carries the equality to refute
· exact hi_ne_n h'
· simp at h12           -- 1 ≠ 2 is trivial
```

### Step 5 — Canonical output guards

The lemma's RHS should expose the **cleaned, post-pruning guards** (e.g.
`c = n-1 ∧ d = n` instead of `1 < c ∧ n = d`). This makes the lemma directly
usable by U3/F1 and U4/F2 via `simpa` — those callers don't need to re-prune.

---

## Minimal-failure-rollback discipline

After Step 1, compile each pruning lemma independently before entering Step 2.
After Step 3, compile the full lemma with `lake env lean <file>`. If it fails:

1. Identify which `hTk` block failed (the error line points to it).
2. Fix only that block — `subst`, `simp`, `rw`, or `omega` adjustments.
3. Recompile — the fix is guaranteed local to that `hTk`.

Do NOT:
- Rewrite multiple `hTk` blocks at once (you won't know which fix worked).
- Switch to a different `simp` or `omega` tactic across all blocks.
- Reorder the `hTk` blocks (order dependence IS the risk this mode avoids).

---

## Recognizing a gate vs a routine lemma

A lemma IS a combinatorial gate when:
- Its proof's natural `split_ifs` count exceeds 4 branches.
- The guard conditions involve independent boolean combinations
  (not a simple `j = c` check that `omega` closes directly).
- Multiple `apply` or `simp` branches produce different goal shapes.
- A reviewer says "this is the only risk point."

A lemma is NOT a gate when it's a single-branch rewrite or a direct `simp`.
These should not receive the full strict-mode treatment.
