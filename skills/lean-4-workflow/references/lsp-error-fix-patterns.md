# LSP Error Fix Patterns for Lean 4 ImageContainment

Common non-mathematical errors encountered when working in the ImageContainment
proof graph, with reproducible fix recipes.

## Forward reference in same file

**Symptom**: `Unknown identifier 'X'` where X is defined later in the same file.

**Cause**: Lean processes top-to-bottom. If lemma A uses lemma B, B must be
defined before A — even in the same `.lean` file.

**Fix**: Move B (and its dependencies) above A. For moves of >5 lines, use
Python via `terminal` (not the `patch` tool — patch requires matching unique
strings which is fragile for block moves):

```python
# Read, extract block, delete, insert, write back
lines = open('file.lean').readlines()
block = lines[start_idx:end_idx]        # extract B (0-indexed)
del lines[start_idx:end_idx]            # remove from original position
lines[target_idx:target_idx] = block    # insert before A
with open('file.lean', 'w') as f: f.writelines(lines)
```

**After every move, verify**:
1. `lake build The.Module` — catches syntax errors immediately
2. Docstring integrity: every `/--` has a closing `-/`, no orphaned docstring
   halves. A split docstring (first half left behind, second half moved) causes
   `unexpected token '/--'; expected 'lemma'` at the orphan site AND
   `unterminated comment` at the second half.
3. Definitional equality: `rw [i = n-1]` on `coeffOf D i (i+1) ...` leaves
   `(n-1)+1` which is NOT `n` — use `rw [hi_eq, show (n-1 : ℕ)+1 = n by omega]`.
4. `linear_combination` variable alignment: if `h_eq` mentions `Y` but goal
   mentions `X` where `X=Y`, `rw [← hX_eq_Y]` (or forward on the goal) first.

**Example**: `boundary_family_right` defined at L1815 but called at L1503 inside
`internal_det_coupling_zero`. 102-line move via Python. Post-move bugs caught:
docstring split in half, `(n-1)+1 ≠ n` definitional mismatch.

## T3 in bracketIdentity always fires for same-row targets

**Symptom**: `omega` can't prove `¬(u = i ∧ j < v)` — because the condition
IS true.

**Cause**: `bracketIdentity_eq_expanded` T3 is `(if u = i ∧ j < v then coeff c d j v else 0)`.
When the target row `u` equals the source row `i` and source col `j` < target col `v`
(usually true for n≥5), T3 FIRES. The code's assumption "only T1 fires" is wrong.

T3 introduces `coeff(partner; j, v)` — a partner-side coefficient that must be
handled separately.

**Fix**: Change `if_neg c3` to `if_pos` with a proper `And` proof, then:
- `simp` the bracketIdentity to expose `goal_coeff + partner_coeff = 0`
- Either prove `partner_coeff = 0` or convert to a `sorry` (0 errors target)

**When to watch for this**: Any bracketIdentity where target row = source row
and target col > source col (e.g., target (2,n) with source (2,3)).

## coeffOf_f rewrite direction

**Symptom**: `rw [coeffOf_f ...] at hb` fails with "Did not find an occurrence".

**Cause**: `coeffOf_f D i j u v ...` rewrites `coeffOf D i j u v` → `D.coeff i j u v`
(in the goal). `hb` from `bracketIdentity_eq_expanded` contains `D.coeff ...`,
NOT `coeffOf ...`. Rewriting `coeffOf_f` AT `hb` finds nothing.

**Fix**: 
- To use `hb` for a `coeffOf` goal: `rw [coeffOf_f ...]` on the GOAL first,
  then use `hb` (which is in `coeff` form).
- To convert `hb` to `coeffOf` form: `rw [← coeffOf_f ...]` at hb.

```lean
-- WRONG: hb contains coeff, not coeffOf
rw [coeffOf_f ...] at hb  -- fails silently

-- RIGHT: convert goal to coeff form, then use hb
rw [coeffOf_f D 2 3 2 (n-1) ...]     -- goal: coeff → D.coeff
-- now exact hb works
```

## `linear_combination` variable mismatch

**Symptom**: `linear_combination h_eq` fails with
`ring failed, ring expressions not equal` — even though `h_eq` looks like it
should close the goal.

**Cause**: `h_eq` mentions variable `Y` (e.g. `coeffOf i n 1 n`) but the goal
mentions variable `X` (e.g. `coeffOf i (i+2) 1 (i+2)`) where `hX_eq_Y : X = Y`
exists. `linear_combination` does NOT use the goal's hypotheses for rewriting —
it only sees `h_eq` and the raw goal.

**Fix**: `rw` first to align variables, then `linear_combination`:

```lean
-- h_eq3: 2*coeffOf i n 1 n = coeffOf i n 1 n       (about Y)
-- hX_eq_Y: coeffOf i (i+2) 1 (i+2) = coeffOf i n 1 n  (X = Y)
-- Goal: (2-1)*coeffOf i (i+2) 1 (i+2) = 0           (about X)

-- WRONG:
rw [hX_eq_Y] at h_eq3   -- NO-OP: h_eq3 mentions Y, not X
linear_combination h_eq3  -- fails: Y vs X

-- RIGHT (option 1): rewrite goal to Y, then use h_eq3
rw [hX_eq_Y]            -- goal becomes about Y
linear_combination h_eq3

-- RIGHT (option 2): rewrite h_eq3 to X
rw [← hX_eq_Y] at h_eq3 -- h_eq3 now about X
linear_combination h_eq3

-- BETTER (direct calc): from 2Y=Y, deduce Y=0, then X=0 via hX_eq_Y
have hY_zero : coeffOf D i n 1 n = 0 := by
  have htemp : (2 : F) * coeffOf D i n 1 n = coeffOf D i n 1 n := ...
  have : ((2 : F) - (1 : F)) * coeffOf D i n 1 n = 0 := by linear_combination htemp
  have : ((2 : F) - (1 : F)) = (1 : F) := by ring
  ...
rw [hX_eq_Y, hY_zero]
```

**Key insight**: `rw [hX_eq_Y] at h_eq` where `h_eq` doesn't contain `X` is a
silent no-op. Use `rw [← hX_eq_Y] at h_eq` to replace `Y` with `X` in the
hypothesis, or `rw [hX_eq_Y]` on the goal to replace `X` with `Y`.

## subst variable capture

**Symptom**: `Unknown identifier 'n'` at the `(n : ℕ)` binder after `subst h_j'_eq_n`.

**Cause**: `subst h` where `h : j' = n` replaces all occurrences of `j'` with `n`,
including in binder annotations like `(n : ℕ)`. This can shadow the module-level
`n`, making subsequent references to `n` ambiguous.

**Fix**: Use `rw [h]` instead of `subst h`. Then explicitly `rw [h]` in the goal
if needed.

```lean
-- WRONG:
subst h_j'_eq_n
-- n is now shadowed

-- RIGHT:
rw [h_j'_eq_n] at hb    -- rewrite in hypothesis
rw [h_j'_eq_n]          -- rewrite in goal
```

## Nat subtraction definitional inequality

**Symptom**: `Type mismatch: coeffOf D (n-3) (n-1) 1 (n-1) = 0` vs
`coeffOf D (n-3) ((n-3)+2) 1 (n-1) = 0`.

**Cause**: In Lean's `Nat`, `(n-3)+1` is NOT definitionally equal to `n-2`,
and `(n-3)+2 ≠ n-1`. The goal and the lemma result use syntactically different
expressions that are equal only by arithmetic.

**Fix**: Use `simpa [show (n-3 : ℕ)+1 = n-2 by omega]` or
`simpa [show (n-3 : ℕ)+2 = n-1 by omega]` to bridge.

```lean
-- a_fire_scc_close returns coeffOf D k (k+1) ...
-- but we need coeffOf D (n-3) (n-2) ...
have := a_fire_scc_close D (n-3) hk2 hk_le_n3
simpa [show (n-3 : ℕ)+1 = n-2 by omega] using this
```

## simp with Ne symmetry

**Symptom**: `simp [hk_ne_1] at hcond` makes no progress when hcond contains
`if 1 = k ∧ ...`.

**Cause**: `hk_ne_1 : k ≠ 1`. `simp` matches `k ≠ 1` against `k = 1 → False`,
but the `if` condition is `1 = k`, not `k = 1`. `simp` doesn't automatically
apply symmetry.

**Fix**: Create the symmetric form: `h1_ne_k : (1 : ℕ) ≠ k` and use that.

```lean
have hk_ne_1 : k ≠ 1 := by omega
have h1_ne_k : (1 : ℕ) ≠ k := by omega  -- symmetric form
simp [h1_ne_k] at hcond  -- works: matches if 1 = k ...
```

## Workflow rule: errors before sorries

When approaching a file with both LSP errors and sorries, clear ALL errors
first — even if that means converting some errors into sorries. A file with
0 errors + N sorries is a stable development surface. A file with M errors
blocks LSP feedback and makes every subsequent edit a blind gamble.

**Recipe**:
1. `lean_diagnostic_messages(severity='error')` → list all errors
2. Classify: forward-ref, type-mismatch, omega-failure, simp-failure
3. Apply fixes from this reference
4. Re-check until 0 errors
5. Only then fill sorries
