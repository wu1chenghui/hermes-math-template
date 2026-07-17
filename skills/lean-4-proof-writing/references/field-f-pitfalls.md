# Field F Pitfalls — Lean 4 Proof Writing

Collected during the 1/2-derivation classification project (June 2026).
These are patterns that repeatedly caused proof failures and wasted time.

---

## `linarith` does NOT work on `Field F`

`linarith` only works on `LinearOrderedCommRing` (ℕ, ℤ, ℚ, ℝ). An arbitrary
`Field F` does not carry a linear order. Using `linarith` on a field equation
fails with `linarith failed to find a contradiction`.

**Pattern: `2y = y → y = 0` over Field F (from D3 internal_det_step, 2026-06)**
```lean4
have hy : coeffOf D i n 2 n = 0 := by
  -- h_eq3: (2 : F) * y = y
  have h_factor : ((2 : F) - 1) * y = 0 := by
    calc
      ((2 : F) - 1) * y = (2 : F) * y - 1 * y := by ring
      _ = y - y := by rw [h_eq3]; simp
      _ = 0 := by ring
  have h_one : ((2 : F) - 1) = (1 : F) := by ring
  rw [h_one] at h_factor
  simpa using h_factor
```
The key: `(2-1) = 1 ≠ 0` in any field with char≠2.  `ring` handles the
algebra; `linarith` is never called.

**Other patterns**:

**Instead**, use one of:
```lean4
-- From -X = 0, get X = 0:
exact neg_eq_zero.mp h

-- From 0 = X, get X = 0:
exact h.symm

-- From X = Y, just use the equality:
exact h

-- From a*X = b*X + c, rearrange manually:
calc
  a*X - b*X = c := by ...
  _ = 0 := by ...

-- For multiplication commutation in fields:
rw [mul_comm]  -- prefer over `ring` which can fail
```

This was the #1 cause of failure in Length3Bridge.lean (7 of 10 lemmas).

---

## `simp` on if-conditions with `¬(P ∧ Q)`

`simp` cannot use `¬(P ∧ Q)` to simplify `if P ∧ Q then A else 0`. It leaves
unsolved goals like `P → A = 0` or `Q → A = 0`.

**Best pattern (discovered June 2026)**: decompose each if-expression into a
separate `have` block that computes its value explicitly, then `simp` the
whole expression at once:

```lean4
-- For each of the four terms A, B, C, D in bracketIdentity:
have hA_zero : (if u < c ∧ v = d then coeff i (i+1) u c else 0) = 0 := by
  simp [hA_false]  -- hA_false : ¬(u < c ∧ v = d)

have hB_val : (if u = c ∧ d < v then coeff i (i+1) d v else 0) = coeff i (i+1) d v := by
  simp [huc_eq, hdv]  -- both conditions true

have hC_zero : (if u = i ∧ i+1 < v then coeff c d (i+1) v else 0) = 0 := by
  simp [hC_false]

have hD_zero : (if u < i ∧ v = i+1 then coeff c d u i else 0) = 0 := by
  simp [hD_false]

-- Then combine:
simp [hA_zero, hB_val, hC_zero, hD_zero] at h_bracket
```

This is superior to `split_ifs with h1 h2` because:
- Each term is handled independently — no 16-case explosion
- `simp` with `¬P` works on a SINGLE if-expression (unlike nested ifs)
- The proof is modular: each `have` block is a one-liner
- Works reliably even with 4 nested if-conditions

**Older pattern (still valid for fewer conditions)**: use explicit `have` blocks for each condition:
```lean4
have h_A_false : ¬ (v = i+N ∧ u < k) := by
  intro h; rcases h with ⟨hveq, _⟩; omega
have h_B_false : ¬ (i = k ∧ i+N < v) := by
  intro h; rcases h with ⟨heq, _⟩; omega
have h_C_true : i = i ∧ k < v := ⟨rfl, h_k_lt_v⟩
simp [h_A_false, h_B_false, h_C_true] at hsplit
```

---

## `omega` fails on nested transitivity constraints

When `omega` cannot handle complex constraint chains, use explicit `calc`:
```lean4
-- Instead of:
have h_k_lt_v : k < v := by omega

-- Use:
have h_k_lt_v : k < v :=\n  calc
    k < i+N := hk_iN
    _ < v := hv_gt_iN
```

This is especially common when you have hypotheses like `hik: i < k`,
`hk_iN: k < i+N`, `hv_gt_iN: i+N < v` and need to prove `k < v`.

## `omega` in `show ... from by omega` — avoid inside `simp`

When using `show ... from by omega` inside a `simp` block, `omega` may
fail because the `simp` context adds unexpected hypotheses. Extract these
to standalone `have` blocks instead:
```lean4
-- Avoid:
simp [show ¬(i+N < v) from by omega, ...] at hsplit

-- Prefer:
have h_not : ¬(i+N < v) := by omega
simp [h_not, ...] at hsplit
```

---

## `simp` needs `hu_lt_i.ne'` explicitly for `u ≠ i` derivations

When you have `hu_lt_i: u < i`, writing `simp [hu_lt_i]` will NOT automatically
derive `u ≠ i`. You need:
```lean4
have hu_ne_i : u ≠ i := by omega
simp [hu_ne_i, ...]
```

Or pass `hu_lt_i.ne'` directly.

---

## `split_ifs` at `h : equation` — sign issues

After `split_ifs` on a coeffOf_cond expansion, the relation between the
original equation and the goal may involve reversals (LHS = RHS vs RHS = LHS).
Always check the direction: `hsplit` may give `2*X = Y` but you need `Y = 2*X`.

```lean4
-- hsplit: 2*coeff = coeff(k)
-- Goal: coeff(k) = 2*coeff
exact hsplit.symm  -- NOT just `exact hsplit`
```

---

## `ring` vs `rw [mul_comm]` on Field F

`ring` works on `CommSemiring` which `Field F` satisfies, but in practice
`rw [mul_comm]` is more reliable for simple commutations:
```lean4
-- Prefer:
rw [mul_comm]
-- Over:
ring
```

---

## `subst` clears variables — prefer `rw`

`subst h` replaces the variable everywhere AND removes it from context.
This destroys hypothesis names (they rewrite their types) and can make
goals unreadable. **Always use `rw [h]` instead of `subst h`**:

```lean4
-- BAD: subst huc_eq; subst hvj
-- → c and v disappear, hc becomes `1 ≤ u`, etc.

-- GOOD:
rw [huc_eq, hvj]
-- → everything still named, just with the substitution applied
```

## `rcases` destroys the original hypothesis — save a copy first

`rcases h` where `h : P ∧ Q` replaces `h` with `<h_left, h_right>`. The
original `h` is GONE. If you need to pass it to another lemma later, save it:

```lean4
-- BAD: rcases hA_cond with ⟨huc, hvd⟩
--       ... later: bracketIdentity_single_active ... hA_cond ...  -- ERROR: not found

-- GOOD: save before rcases
have hA_saved := hA_cond
rcases hA_cond with ⟨huc, hvd⟩
-- ... later: bracketIdentity_single_active ... hA_saved ...
```

Or pass the reconstructed pair: `⟨huc, hvd⟩`.

---

## Dot notation breaks `rw` — prefer `simpa`

When `coeffOf D i j u v` is written in dot notation as `D.coeffOf i j u v`,
`rw [coeffOf_f D i j u v ...]` may fail because it cannot find the pattern.
The LHS of `coeffOf_f` is `coeffOf D i j u v` but the goal displays
`D.coeffOf i j u v`.  **Solution**: use `simpa [hcoeff1, hcoeff2]` instead
of `rw [hcoeff1, hcoeff2]`.

```lean4
-- BAD (may fail with dot notation):
have hcoeff : coeffOf D i (i+1) d v = D.coeff i (i+1) d v := coeffOf_f ...
rw [hcoeff] at h_bracket ⊢
exact h_bracket

-- GOOD (always works):
have hcoeff : coeffOf D i (i+1) d v = D.coeff i (i+1) d v := coeffOf_f ...
simpa [hcoeff] using h_bracket
```

This is especially common with `coeffOf_f` conversions.  The `simpa` pattern
is robust regardless of whether Lean displays the term in dot notation.

## `half_leibniz` requires validity hypotheses — never call without them

The `HalfDerivation.half_leibniz` field has type:
```lean4
half_leibniz (i j k l u v : ℕ) (hi : 1 ≤ i) (hij : i < j) (hjn : j ≤ n)
  (hk : 1 ≤ k) (hkl : k < l) (hln : l ≤ n) (hu : 1 ≤ u) (huv : u < v) (hvn : v ≤ n) :
  2 * bracketSource coeff i j k l u v = bracketIdentity coeff i j k l u v
```

The 9 validity hypotheses MUST be provided. Calling `D.half_leibniz i j c d u v`
without them leaves an unsolved goal `case hi : 1 ≤ i`.

**Fix**: either add the hypotheses to your theorem, or use `coeffOf` wrappers
that handle bounds checking internally.

```lean4
-- CORRECT: include all validity hypotheses
theorem my_lemma (D : HalfDerivation F n) (i j c d u v : ℕ)
    (hi : 1 ≤ i) (hij : i < j) (hjn : j ≤ n)
    (hc : 1 ≤ c) (hcd : c < d) (hdn : d ≤ n)
    (hu : 1 ≤ u) (huv : u < v) (hvn : v ≤ n) : ... := by
  have h := D.half_leibniz i j c d u v hi hij hjn hc hcd hdn hu huv hvn
```

---\n\n## Set-builder variable collision with context binders

When `variable {n : ℕ}` is in scope, set-builder notation like
`{(i, i+1) | 1 ≤ i ∧ i+1 ≤ n}` produces "Invalid pattern variable: `i` was
already used" because `i` is already a binder in the current context (from
a theorem parameter or variable declaration).

**Fix**: use a fresh variable name in the set-builder:
```lean4
-- BAD:
def active_sources : Set (ℕ × ℕ) := {(i, i+1) | 1 ≤ i ∧ i+1 ≤ n}
-- ERROR: Invalid pattern variable: Variable name `i` was already used

-- GOOD:
def active_sources : Set (ℕ × ℕ) := {(src, src+1) | 1 ≤ src ∧ src+1 ≤ n}
```

---\n\n## `simp` with `CharNeTwo` can simplify `2*x = 0` to `x = 0`

When `CharNeTwo F` is in scope, `simp` may apply `mul_eq_zero` with the
knowledge that `2 ≠ 0`, silently converting `2*x = 0` to `x = 0`. This
changes the type of a hypothesis unexpectedly, causing downstream `rw`
or `apply` failures.

**Fix**: use `simpa` to create a new hypothesis rather than `simp` at the
original one.
```lean4
-- AVOID (simp changes h_cond type):
simp at h_cond
-- h_cond: 2*x = 0  →  h_cond: x = 0  (unexpected!)

-- PREFER:
have h_cond' : 2 * coeffOf D i (i+2) u v = 0 := by
  simpa using h_cond
-- h_cond unchanged, h_cond' has the simplified RHS
```

---\n\n## `sub_self` not `sub_zero` for zero-subtraction in fields

`sub_zero a` gives `a - 0 = a`. `zero_sub a` gives `0 - a = -a`.
To get `a - a = 0`, use `sub_self a`.

```lean4
-- Getting 2*x - 2*x = 0:
-- WRONG: rw [sub_zero]  -- gives 2*x - 0 = 0
-- CORRECT:
rw [← h]  -- replace 2*bracketSource with bracketIdentity
-- goal becomes: bracketIdentity - bracketIdentity = 0
exact sub_self _
```

---\n\n## I_target_set membership proofs — use helper lemmas with omega

Direct `simp [I_target_set]` on a membership goal `(u, v) ∈ I_target_set hn`
expands to checking `(u,v) = (1,n-1) ∨ (u,v) = (1,n) ∨ (u,v) = (2,n)`.
For NEGATIVE membership `(u,v) ∉ I_target_set hn`, create helper lemmas:

```lean4
lemma not_I_target_of_col_lt_n1 (hn : 3 ≤ n) (a b : ℕ) (hb : b < n-1) :
    (a, b) ∉ I_target_set hn := by
  intro hmem; simp [I_target_set] at hmem
  rcases hmem with (⟨_, h⟩ | ⟨_, h⟩ | ⟨_, h⟩)
  · omega; · omega; · omega

lemma not_I_target_of_row_ge_3 (hn : 3 ≤ n) (a b : ℕ) (ha : 3 ≤ a) :
    (a, b) ∉ I_target_set hn := by
  intro hmem; simp [I_target_set] at hmem
  rcases hmem with (⟨h, _⟩ | ⟨h, _⟩ | ⟨h, _⟩)
  · omega; · omega; · omega
```

This avoids repeating the `simp` + `rcases` + `omega` block 4+ times per lemma.

---

When the goal contains `A - B = 0` but `h_bracket` gives `A + (-B) = 0`,
the two forms are definitionally distinct in Lean.  Use:

```lean4
simpa [sub_eq_add_neg] using h_bracket
```

This avoids the need to `linarith` (which fails on Field F).

---

## Over-claiming theorem statements — the P_1 counterexample trap

When a theorem's induction creates width-1 child sources, the theorem statement
must be checked against KNOWN counterexamples for those specific child targets.
**Don't blindly strengthen the statement to cover the base case** — first verify
what the induction actually needs.

**Classic case (June 2026)**: `coeffOf_nonadjacent` claimed `c_{ij}^{uv} = 0` for
ALL sources including adjacent. But P_1(E_{12}) = E_{1n} is a valid 1/2-derivation
with c_{12}^{1n} = 1 ≠ 0. The correct split: nonadjacent sources → all offdiag = 0;
adjacent sources → offdiag = 0 EXCEPT at (1,n).

For the methodology to check whether a counterexample contaminates an induction,
see `references/induction-contamination-check.md`.

---

## `#eval!` needed for `lean --run` when file has `sorry`

Use `#eval!` (not `#eval`) at the top level of `--run` files.
`#eval` aborts if the file contains any `sorry` (even in unused theorems).
`#eval!` forces evaluation regardless.

```bash
lake env lean --run E/Analysis.lean  # file must use #eval!
```

---

## Lean 4 forward references — definitions must come BEFORE use

Unlike some languages, Lean 4 does NOT allow forward references outside `mutual`
blocks.  A `lemma` defined at line 500 cannot be called by a `theorem` at line 312.

**Fix**: place helper lemmas BEFORE the theorems that call them.  If you must
have a forward reference, extract the shared logic into a `mutual` block, or
restructure the file so helpers come first.

This is especially important for "step lemmas" (like `internal_det_step`) that
are called from within large induction bodies.

### `String.repeat` and `String.pushn` may not exist

In Lean 4.31-rc1, `String.repeat` and `String.pushn` may be missing. Use:
```lean4
-- Instead of: " ".repeat n
String.join (List.replicate n " ")

-- Instead of: " ".pushn n
String.join (List.replicate n " ")
```

### `Module` is a reserved name (Category Theory)

`Module` conflicts with `Module R M` from mathlib's category theory.
When creating an inductive type for analysis tools, use a different name:
```lean4
-- BAD:
inductive Module where | bridge | only | diag

-- GOOD:
inductive FMod where | bridge | only | diag
```

### `|>.length` pipe operator may not be available

The `|>` forward pipe works but `|>.field` projection may fail in 4.31:
```lean4
-- BAD:
all.filter (λ s => hasMod s m) |>.length

-- GOOD:
(all.filter (λ s => hasMod s m)).length
```

### `List.bind` and `List.groupBy` not available in 4.31

Lean 4.31-rc1 does NOT have `List.bind` or `List.groupBy`. Common patterns
that rely on these need manual alternatives.

**`List.bind`**: use `List.join ∘ List.map` or explicit nested loops:
```lean4
-- BAD:
List.range (n+1) |>.bind λ u =>
  List.range (n+1) |>.bind λ v => ...

-- GOOD (for #eval! / IO):
for u in List.range (n+1) do
  for v in List.range (n+1) do
    if cond u v then ...
```

**`List.groupBy`**: use manual filters:
```lean4
-- Instead of: ps.groupBy (λ a b => a.className = b.className)
let commuting := ps.filter (λ p => p.className = "Commuting") |>.length
let only := ps.filter (λ p => p.className = "ONLY") |>.length
```

### Structure field access in `#eval!` blocks may fail (4.31-rc1)

In Lean 4.31-rc1, `structure Foo where a b c d : ℕ` may NOT produce usable
field projections inside `#eval!` or `#eval` IO blocks — the error is
"Invalid field notation: Field projection operates on types of the form C ..."
even though the structure is correctly defined at module level.

**Workaround**: use plain tuples instead of structures in experiment files:
```lean4
-- BAD (field access fails in #eval!):
structure Node where
  i j u v : ℕ
-- later: nd.i  →  ERROR: Invalid field notation

-- GOOD (tuple, works reliably in #eval!):
def children (i j u v : ℕ) : List (String × ℕ × ℕ × ℕ × ℕ) := ...
-- destructure with: for (_,_,i',j',u',v') in children ...
```

This only affects self-contained `#eval!` experiment files — `.lean` files
compiled via `lake build` do NOT have this issue.

### `Nat.min_eq_or_eq_left` does not exist in 4.31

Use `omega` or explicit case analysis on `Nat.min` instead:
```lean4
-- BAD:
rcases Nat.min_eq_or_eq_left a b with (h | h)

-- GOOD (use omega for inequalities):
have h_lt : min (k - i) (j - k) < j - i := by
  omega
```

`omega` is generally preferred over explicit `Nat.sub_lt_sub_left` /
`Nat.sub_lt_sub_right` for width inequalities — the lemma signatures
have counterintuitive argument ordering in this version.

### Inductive constructors with default parameter values are invalid

Lean 4 does not support `:=` default values in inductive constructor parameters:
```lean4
-- BAD (parse error):
inductive Child where
  | active_A (i j k u : ℕ) (hvj : v := j)

-- GOOD (all parameters explicit):
inductive Child where
  | active_A (i j k u : ℕ) (huk : u < k)
```

## `s!` interpolation with format specs like `{name,-15}` can produce
`expected '}'` errors in 4.31-rc1. Use explicit padding:
```lean4
def padRight (s : String) (n : Nat) : String :=
  s ++ String.join (List.replicate (n - s.length) " ")

-- Then:
padRight s.name 15 ++ " |" ++ ...
```

## `dsimp` can't reduce named-argument applications — use `simp only [defn]`

When a definition is called with explicit named arguments like
`coeff_A (F := F) (n := n) i`, `dsimp` makes no progress — it can't
pattern-match through the named binder syntax:

```lean4
-- BAD: "dsimp made no progress"
unfold bracketSource bracketIdentity
dsimp

-- GOOD:
unfold bracketSource bracketIdentity bracketLeft bracketRight
simp only [coeff_A]
```

This is especially common when definitions are made under `variable (F) (n)`
(explicit) but called from `variable {F n}` (implicit) scopes, requiring
named-argument application.

**Rule**: after `unfold`-ing the outer formulas, use `simp only [defn_name]`
(not `dsimp`) to expand the coefficient definition.

---

## `simp` DOES handle `¬` hypotheses (no `≠` conversion needed)

Contrary to initial intuition, `simp [hb]` where `hb : ¬ b = n` DOES
simplify `if b = n then X else Y` to `Y`:

```lean4
example (b n : ℕ) (hb : ¬ b = n) (x y : ℕ) : (if b = n then x else y) = y := by
  simp [hb]  -- works!
```

`by_cases h : P` gives `h : ¬ P` in the false branch, and this `¬ P` is
definitionally equal to `P → False`. `simp` can use it as a rewrite rule.

**However**: `simp [hb]` may leave residual goals when `¬(P ∧ Q)` appears
inside a nested `if` (see "simp on if-conditions" above). For single-condition
`if`s, `simp` with a `¬` hypothesis works directly.

**Do NOT** waste time converting `¬ b = n` to `b ≠ n` via `λ h => hb h`:
they are definitionally equal and `simp` handles both identically.

---

## `split_ifs` timeout — even at high heartbeat, 12+ conditions explode

`split_ifs` with 12+ remaining `if` conditions creates O(2^12)=4096 subgoals.
This times out even at 1,000,000 heartbeats. The `by_cases` chain approach
(already documented above) is NOT just a style preference — it's REQUIRED
for cases with many residual conditions.

**Symptom**: `(deterministic) timeout at whnf, maximum number of heartbeats
(1000000) has been reached` at the `split_ifs` line.

**Fix**: pre-split with `by_cases` on the highest-impact conditions (source
activity, target matching) before using `split_ifs` on the remaining few:

```lean4
-- Pre-split on whether source-1 and source-2 are "active" (match coeff domain)
by_cases ha : a = n-1
· by_cases hb : b = n
  · -- first source active
    by_cases hc : c = n-1
    · by_cases hd : d = n
      · -- both sources active → ~4 remaining conditions → split_ifs safe
        simp [ha, hb, hc, hd]; split_ifs <;> try ring <;> try omega
      · -- c=n-1, d≠n → contradiction
        exfalso; apply hd; omega
    · -- c≠n-1, first source only active
      simp [ha, hb]; split_ifs <;> try ring <;> try omega
  · -- a=n-1, b≠n → first source not active
    ...
· -- a≠n-1 → first source not active
  ...
```

After the `by_cases` splits, `simp` with the source-activity hypotheses
eliminates most coeff-level conditions, leaving only ~4-6 bracket-level
conditions for `split_ifs` — safe.

### `apply if_neg; omega` — clean pattern for if-expressions with ∧ conditions

When you need to prove `(if P ∧ Q then A else 0) = 0` and `P ∧ Q` is false
in context, `simp` with `¬(P∧Q)` often leaves residual goals. The cleanest
pattern is:

```lean4
-- For false condition:
have hA24 : (if (2:ℕ) < 3 ∧ n = 4 then coeffOf D 2 3 2 3 else 0) = 0 := by
  apply if_neg; omega

-- For true condition:
have hC24_true : (if (2:ℕ) = 2 ∧ 3 < n then coeffOf D 3 4 3 n else 0) = coeffOf D 3 4 3 n := by
  apply if_pos; exact ⟨rfl, by omega⟩
```

`if_neg` expects `¬condition` and rewrites the if to the else branch.
`if_pos` expects `condition` and rewrites to the then branch. Both are
one-liners and never leave residual goals, unlike `simp` or `split_ifs`.

This is especially useful inside `coeffOf_cond_of` expansions where each
of the A/B/C/D terms has a conjunction condition.

### `ring` fails on `-(-x) = x` in generic Field F — use `simp [neg_neg]`

`ring` works on polynomial equations (`+`, `*`, `^`), NOT on unary `-` patterns.
For `-(-x) = x`, use:

```lean4
-- BAD: ring → unsolved goal x*2 = -(x*2) (ring misinterprets −)
calc
  D.coeff 3 4 3 n = -(-(D.coeff 3 4 3 n)) := by ring  -- FAILS

-- GOOD:
  _ = -(-(D.coeff 3 4 3 n)) := by simp [neg_neg]
```

Or use `congrArg Neg.neg` to negate both sides of an equation directly:

```lean4
-- hhl: 2·(-x) = -y
-- Want: y = 2x
have htemp := congrArg (fun (t : F) => -t) hhl
-- htemp: -(2·(-x)) = y  →  2x = y
simp at htemp
exact htemp.symm
```

This pattern avoids calc blocks entirely when the algebra is just
negating both sides and simplifying.

### `mul_right_cancel₀` cancels on the RIGHT — need `mul_comm` for left factors

`mul_right_cancel₀ h2 : a*2 = b*2 → a = b` cancels the factor on the RIGHT.
When your equation has the factor on the LEFT (`2*a = 2*b`), you need `mul_comm`:

```lean4
have h2 : (2 : F) ≠ 0 := CharNeTwo.char_ne_two
apply mul_right_cancel₀ h2
-- Goal: coeffOf ... * 2 = coeffOf ... * 2
-- But we have: 2 * coeffOf ... = 2 * coeffOf ...
simpa [mul_comm] using hcond24
```

Alternatively, avoid `mul_right_cancel₀` and use direct algebra:

```lean4
-- From 2x = 2y and 2≠0, get x=y:
have : (2 : F) * (x - y) = 0 := by rw [mul_sub, hcond24, sub_self]
have hsub : x - y = 0 := (mul_eq_zero.mp this).resolve_left h2
exact sub_eq_zero.mp hsub
```

The direct algebra approach is more verbose but avoids the `mul_comm` dance.

---

## `exfalso; apply hd; omega` — contradiction from `by_cases` false branch

When a `by_cases hd : d = n` false branch is reached but all known constraints
force `d = n` (e.g., `c = n-1`, `c < d`, `d ≤ n`), use this pattern:

```lean4
· -- hd : ¬ d = n, but c=n-1 and c<d≤n forces d=n
  exfalso; apply hd; omega
```

**Not**: `have hd_eq_n : d = n := by omega; exact hd hd_eq_n` — this fails
with "type mismatch" because the goal is the bracket equation, not `False`.
`exfalso` changes the goal to `False`, then `apply hd` replaces it with `d = n`,
which `omega` solves.

---

## `half_leibniz` proof pattern — 4-branch source-activity tree

When proving `half_leibniz` for a specific coefficient function (e.g., `coeff_E`
mapping `E_{n-1,n} → E_{1,n-1}`), the standard structure is a 4-branch tree
based on whether each source pair matches the coeff's active domain.

**Pattern** (for a coeff with active source `(n-1, n)`):

```lean4
unfold bracketSource bracketIdentity bracketLeft bracketRight
simp only [coeff_E]
-- Branch 1: both sources active
by_cases ha : a = n-1
· by_cases hb : b = n
  · -- first source = E_{n-1,n}
    by_cases hc : c = n-1
    · by_cases hd : d = n
      · -- BOTH sources active
        -- bracketLeft + bracketRight cancel; bracketSource = 0
        simp [ha, hb, hc, hd]; split_ifs <;> try ring <;> try omega
      · -- c=n-1, d≠n: c<d≤n forces d=n → contradiction
        exfalso; apply hd; omega
    · -- ONLY first source active
      simp [ha, hb]; split_ifs <;> try ring <;> try omega
  · -- a=n-1, b≠n → first source NOT active
    by_cases hc : c = n-1
    · by_cases hd : d = n
      · -- ONLY second source active
        simp [ha, hb, hc, hd]; split_ifs <;> try ring <;> try omega
      · exfalso; apply hd; omega
    · -- NEITHER source active (hb:¬b=n, hc:¬c=n-1)
      -- split_ifs WITHOUT prior simp → TIMEOUT
      -- INSTEAD: simp first then split_ifs
      simp [ha, hb, hc]; split_ifs <;> try ring <;> try omega
· -- a≠n-1 → first source NOT active
  by_cases hc : c = n-1
  · by_cases hd : d = n
    · -- ONLY second source active
      simp [ha, hc, hd]; split_ifs <;> try ring <;> try omega
    · exfalso; apply hd; omega
  · -- NEITHER source active
    simp [ha, hc]; ring
```

**Key principles**:

1. **Split on source activity first** (does each source match the coeff's domain?).
   This eliminates 8-10 inner `if` conditions before `split_ifs`.

2. **"Both active" branch**: bracketLeft and bracketRight cancel (opposite signs on
   the same non-zero coefficient). `split_ifs` on just ~4 bracket-level conditions.

3. **"One active" branches**: `simp` with the active-source equality eliminates coeff
   calls; `split_ifs` on remaining bracket conditions.

4. **"Neither active" branch (hardest)**: Both sources fail the coeff's domain check.
   Use `simp [ha, hb, hc]` BEFORE `split_ifs` — the `¬` hypotheses simplify inner
   `if b=n` conditions, leaving only bracket-level conditions. Then `split_ifs`.

5. **Contradiction sub-branches**: `c=n-1, d≠n` implies `n-1 < d ≤ n` → `d=n`. Use
   `exfalso; apply hd; omega` (NOT `have hd_eq := ...; exact hd hd_eq`).

6. **Never call `split_ifs` without prior `simp`** in a branch with `¬` hypotheses.
   The full `split_ifs` on ~15 conditions creates 32768+ subgoals and times out
   even at 1M heartbeats.

**Adapting for coeffs with multiple active sources** (B, C, D, F):
- For B (source `(1,2)` → `(2,n)`): same structure, `(n-1,n)` → `(1,2)`.
- For C (two active sources): need two sets of `by_cases` (is it `(1,3)`? is it `(2,3)`?).
- For D, F: similar multi-source branching.

**Preferred over**: writing a general lemma about `bracketSource = 0` for adjacent
sources. The structural lemma approach (Option A) is the long-term solution but
the `by_cases` tree is the fastest path to a working proof for each coeff type.

---

## `¬` conversion trap — do NOT waste time converting `¬(a=b)` to `a ≠ b`

`by_cases h : P` in the false branch gives `h : ¬ P`. `¬ P` is `P → False`, and
`a ≠ b` is also `a = b → False`. They are **definitionally equal** in Lean 4.

```lean4
-- UNNECESSARY (wastes a line; type error possible in nested scopes):
have hbn : b ≠ n := λ h => hb h

-- CORRECT (simp works directly with ¬):
simp [hb]
```

`simp [hb]` where `hb : ¬ b = n` simplifies `if b = n then X else Y` to `Y`
without any conversion. The `λ h => hb h` pattern is unnecessary and can fail
with "Function expected" if `hb` is actually `b = n` (true branch) rather than
`¬ b = n` (false branch). This confusion is especially easy in deeply nested
`by_cases` trees where the branch semantics are non-obvious.

**Rule**: Always check WHICH branch of `by_cases` you're in before using `hb`.
In the `·` (true) branch, `hb : b = n` (equality). In the `·` (false) branch
that follows, `hb : ¬ b = n`. The two bullets look identical but `hb` has
opposite types.

---

## `split_ifs` explosion — use `by_cases` chain instead

`split_ifs with hi hij hjn hu huv hvn` on a 6-layer nested if (like `coeffOf`)
creates 2^6=64 cases, many of which have "No goals to be solved". The syntax
`· rfl; · rfl; ...` in a single arm can't target sub-arms correctly.

**Fix**: replace `split_ifs` with a `by_cases` chain:

```lean4
-- BAD (64 cases, unmaintainable):
unfold coeffOf
split_ifs with hi hij hjn hu huv hvn
· exfalso; exact h ⟨hi, hij, hjn⟩
· rfl; · rfl; · rfl; · rfl; · rfl; · rfl

-- GOOD (6 cases, linear path):
unfold coeffOf
by_cases hi : 1 ≤ i
· by_cases hij : i < j
  · by_cases hjn : j ≤ n
    · exfalso; exact h ⟨hi, hij, hjn⟩
    · simp [hjn]
  · simp [hij]
· simp [hi]
```

**Rule**: `split_ifs with N names` creates 2^N cases. For N > 3,
use `by_cases` chain or `simp` with relevant hypotheses. The `by_cases`
chain is also more readable — each branch is a single path through
the if-stack.

---\n\n## `rw` direction trap: converting coeffOf ↔ D.coeff\n\nWhen `h_cond` has `D.coeff` on LHS and the goal has `coeffOf`, the naive\n`rw [coeffOf_f ...] at h_cond` fails because `coeffOf_f` gives\n`coeffOf = D.coeff` but `h_cond` contains `D.coeff`, not `coeffOf`.\n\n**Fix**: use `←` to rewrite in the opposite direction:\n\n```lean4\n-- h_cond: 2 * D.coeff i j u v = (if...D.coeff...) - ...\n-- Goal:   2 * coeffOf D i j u v = (if...coeffOf...) - ...\n\n-- BAD (finds nothing):\nhave h_lhs : coeffOf D i j u v = D.coeff i j u v := coeffOf_f ...\nrw [h_lhs] at h_cond     -- tries to find `coeffOf` in h_cond → fails\n\n-- GOOD:\nrw [← h_lhs] at h_cond    -- replaces `D.coeff` with `coeffOf`\n```\n\n**Rule**: `rw [h]` looks for the LHS of `h` in the target. `rw [← h]`\nlooks for the RHS. When converting from raw `D.coeff` to the `coeffOf`\nwrapper, the raw coeff is already present → use `←`.\n\n---\n\n## `rw at *` is DANGEROUS — rewrites in ALL hypotheses including previously defined helpers

`rw [h] at *` rewrites in EVERY hypothesis AND the goal. This includes
carefully crafted helper lemmas defined just above. After `rw at *`, those
helpers' types change (sometimes becoming ill-typed), and subsequent `rw`
or `simpa` using them fails with "Did not find an occurrence of the pattern"
because the pattern they describe no longer exists.

**ALWAYS prefer explicit targets**: `rw [h] at hcond ⊢` (only hcond and goal).
Never use `at *` unless you have exactly one rewrite and no dependent hypotheses.

**Real example (2026-06-26, ImageContainment.lean B3)**: 
```lean4
-- BAD: rw [hk1, hm_val] at *
-- → rewrites in hA, hBchild, hC, hD — their if-expressions now reference
--   explicit values that no longer match hcond's expressions
-- → rw [hA, hBchild, hC, hD] at hcond fails

-- GOOD: rw [hk1, hm_val] at hcond ⊢
-- → only transforms hcond and the goal, leaves helpers untouched
```

**Pattern**: define helpers AFTER rewrites, or rewrite only specific targets.
When you must rewrite before defining helpers, use `rw at hcond ⊢`.

---

## `set` let-in binder — `rw` cannot match, use `dsimp` or `rw [hm]`

`set m := k + 1 with hm` creates a `let` binder `m` and a hypothesis
`hm : m = k + 1`. The `let` binder is NOT a definitional abbreviation —
`rw [hm_val]` where `hm_val : m = 2` will NOT find `m` in hcond because
`m` is a let-in, not a free variable.

**Fix**: use `dsimp [m] at hcond` to unfold the let-in, OR use
`rw [hm] at hcond` to replace `m` with `k+1` (using the `hm` from `set`).

**After `rw [hk1]` (replacing `k` with `1`)**: `dsimp [m]` expands `m` to
`1+1`, which `simp` then normalizes to `2`. Or `rw [hm]` replaces `m` with
`k+1` = `1+1`.

**Best practice**: avoid `set` in contexts where you need to rewrite over
the binder. Use `let m := k+1` or just inline `k+1` directly.

---

## Rebuild `coeffOf_cond_of` from scratch instead of transforming hcond

When hcond comes from `coeffOf_cond_of` with symbolic indices (k, m=k+1)
and you later need a version with concrete indices (k=1, m=2), do NOT try
to `rw [hk1, hm_val]` + `dsimp [m]` + `simp` the existing hcond. The
chain of transformations frequently gets stuck on let-in binders, `1+1 ≠ 2`
normalization, and if-expression syntactic mismatches.

**Instead**: call `coeffOf_cond_of` AGAIN with the concrete indices:
```lean4
-- BAD (fragile, 5+ iterations):
rw [hk1] at hcond ⊢; dsimp [m] at hcond; simp [hB.2] at hcond
-- simpa using hcond  -- STILL fails: type mismatch after simplification

-- GOOD (clean, works first time):
have hcoeff := coeffOf_cond_of D 1 j' 2 n (by omega) (by omega) (by omega)
  (by omega) (by omega) (le_refl n) 2 (by omega) (by omega)
simp [hj'_lt_n] at hcoeff
-- hcoeff: 2·coeffOf(1,j';2,n) = -coeffOf(1,2;j',n)
rw [hchild] at hcoeff; simpa [neg_zero] using hcoeff
```

This avoids ALL the `set`/`rw`/`dsimp`/`simp` issues because the new call
uses concrete indices from the start. The cost is one extra `coeffOf_cond_of`
call — negligible compared to the debugging time saved.

**Rule**: when the index transformations get to 3+ steps, stop and rebuild
from scratch with concrete indices.\n\nWhen a hypothesis has `(A - B) + (C - D)` but the goal has\n`A - B + C - D` (different parenthesization), `exact h` or `simpa` will\nfail even though the expressions are equal by associativity.\n\n```lean4\n-- h_half: 2*coeff = (A - B) + (C - D)\n-- Goal:   2*coeff = A - B + C - D\n\n-- BAD (type mismatch):\nexact h_half\n\n-- GOOD:\nsimpa [add_assoc, sub_eq_add_neg] using h_half\n```\n\n`simpa` with `add_assoc` and `sub_eq_add_neg` normalizes both sides\nto `A + (-B) + C + (-D)`, making them definitionally equal.\n\n---\n\n## Orphaned doc comment before `end` causes parse error\n\nA `/-- ... -/` doc comment that is NOT immediately followed by a\ndeclaration (theorem, def, lemma) causes `unexpected token 'end'`:\n\n```lean4\n-- BAD:\n/-- This lemma is deferred. -/\n-- TODO: implement later\n\nend MyNamespace\n\n-- GOOD (remove orphaned doc comment, or convert to plain comment):\n-- This lemma is deferred.\n-- TODO: implement later\n\nend MyNamespace\n```\n\n**Rule**: any `/-- ... -/` or `/-! ... -/` must be immediately followed\nby a declaration. Stray doc-comments (even before a `--` line comment)\nwill confuse the parser.\n\n---\n\n## `variable (F)` vs `variable {F}` — explicit binders poison all downstream calls

When `F` and `n` are declared as `variable (F : Type) ... (n : ℕ)` (explicit),
every `def` and `theorem` defined under them gets `F` and `n` as EXPLICIT
binder arguments.  Later, inside a `variable {F n}` (implicit) section,
calling those definitions requires `(F := F) (n := n)` named arguments:

```lean4
-- BAD (fails with "Application type mismatch"):
def T (D : NnDerivation F n) : NnDerivation F n where
  coeff i j u v := ... scalarCoeff F n D ...

-- GOOD:
def T (D : NnDerivation F n) : NnDerivation F n where
  coeff i j u v := ... scalarCoeff (F := F) (n := n) D ...
```

**Rule**: When a definition is made under `variable (F ...) (n ...)` (explicit),
all downstream calls from `variable {F n}` (implicit) sections MUST use named
arguments.  Alternatively, define everything under `variable {F n}` (implicit)
from the start, or use `variable (F n)` (explicit) everywhere consistently.

This caused 5+ build failures in the Classification layer refactoring.

When creating `#eval!`-based enumeration/analysis files, do NOT import
project `.lean` modules (like `E.Bracket`, `E.HalfDerivation.ConstantChain`).
These require `.olean` files that may not exist if the module isn't in
the `E.lean` import tree.

Instead, use only `import Mathlib.Tactic` and define self-contained
data structures and logic. Run with `lake env lean --run`.

```lean4
-- GOOD (self-contained analysis file):
import Mathlib.Tactic

inductive FMod where | bridge | only | ...

#eval! show IO Unit from do
  IO.println "analysis output"

-- Run: lake env lean --run E/Analysis.lean
```

## `bracket_none` needs `¬ b = a`, not `a ≠ b` — wrap with `Ne.symm`

`bracket_none` from `MatrixBasis` expects `(x.j ≠ y.i)` and `(y.j ≠ x.i)`.
`Nat.ne_of_lt h` gives `a ≠ b` (i.e., `¬ a = b`), but `bracket_none` wants
`¬ b = a`. Use `Ne.symm`:

```lean4
-- h: y.i < n
have hy_i_ne_n : y.i ≠ n := Nat.ne_of_lt h
-- bracket_none expects ¬ n = y.i, NOT ¬ y.i = n
apply bracket_none
· simpa [gen_2_n] using Ne.symm hy_i_ne_n  -- converts y.i ≠ n → n ≠ y.i
· ...
```

**Similarly for `¬` propositions**: `hy_i : ¬ y.i = n-1` — to get `¬ n-1 = y.i`,
use `Ne.symm hy_i`, NOT `hy_i.symm` (`.symm` is a field of `Eq`, not `¬`).

---

## `open MatIdx` silently shadows ℕ binders `i`/`j` — elaboration resolves to `MatIdx.i` (function type)

**Symptom**: Errors like:
```
don't know how to synthesize implicit argument `α`
  @Eq (MatIdx ?m.716 → ℕ) i 1
```
The key diagnostic: `i` is being elaborated as `MatIdx → ℕ` (a function), NOT as `ℕ`.
Any tactic that needs `i = 1` — `omega`, `rw`, `linarith`, even `have hi1 : i = 1 := ...` — fails
because the GOAL TYPE itself resolves `i` as `MatIdx.i`.

**Root cause**: `open MatIdx` at the top of the file (or in an imported file that re-exports)
exposes `MatIdx.i` and `MatIdx.j` as projection functions. Binders named `i` or `j` in
`(i j u : ℕ)` become ambiguous — the elaborator may pick `MatIdx.i` over the binder.

**Detection**: Before debugging any proof in a file with `i`-related elaboration errors:
```bash
grep "open MatIdx" <file>.lean
```
If the file itself has `open MatIdx`, and never uses `MatIdx.X` syntax, remove it.

**Fix**: Remove `MatIdx` from the `open` line:
```lean4
-- BEFORE:
open MatIdx PhiOperator HalfDerivation ...

-- AFTER:
open PhiOperator HalfDerivation ...
```

Only `open MatIdx` if the file explicitly uses `MatIdx.ext_iff`, `MatIdx.len`,
or destructures `MatIdx` values. Most coefficient-level files do NOT need it —
they work with bare `ℕ` indices.

**Why `lake build` can pass while the file won't compile standalone**:
`lake build` caches `.olean` files. If `MatIdx` is removed from open and `lake build E.Classification.Centering`
fails but full `lake build` passes, stale `.olean` files from dependencies are masking the error.
Always run full `lake build` after changing `open` directives.

**Related pitfall**: `omega` cannot see through `MatIdx` fields (see separate entry above).
The two are different issues — that one is about omega not accessing structure fields;
this one is about the binder name itself being hijacked.

`omega` only sees linear arithmetic on `Nat` and `Int`. It cannot access
fields of a `structure MatIdx` like `.hpos`, `.hlt`, `.hle` unless they
are in the hypothesis context.

```lean4
-- BAD: omega fails with "No usable constraints found"
have hy_j_ge_2 : 2 ≤ y.j := by omega

-- GOOD: extract fields first, then use omega
have hy_j_ge_2 : 2 ≤ y.j := by
  have hpos : 1 ≤ y.i := y.hpos
  have hlt : y.i < y.j := y.hlt
  omega
```

**Rule**: before `omega` on any goal involving `MatIdx` fields, write
`have` blocks extracting `y.hpos`, `y.hlt`, `y.hle` into the context.

---

## `MatIdx.ext_iff.mpr` for structure equality from field equalities

When you need to prove `y = gen_n1_n hn` (both `MatIdx n`), use:

```lean4
have hy_eq : y = gen_n1_n hn := by
  apply (MatIdx.ext_iff _ _).mpr
  exact ⟨hy_i_eq, hy_j_eq⟩  -- pair of field equalities
```

`MatIdx.ext_iff a b` returns `a = b ↔ a.i = b.i ∧ a.j = b.j`. Use `.mpr` to
convert the field equalities into a structure equality.

---

## `simp` unused argument warnings — remove from simp list

```lean4
-- WARNING: This simp argument is unused: hn
simp [generators, hn]

-- FIX: remove unused args
simp [generators]
```

`simp` warns when arguments in the list aren't used in the simplification.
These are non-fatal in `lake build` but clutter the output. Remove them.

---

When using `CharNeTwo` to divide by 2, expressions like `(1/2)*λ + (1/2)*λ`
don't simplify to `λ` with `field_simp` alone. You need both tactics:

```lean4
have hchar : (2 : F) ≠ 0 := CharNeTwo.char_ne_two
-- Goal: (1/2)*λ + (1/2)*λ = λ
field_simp [hchar]
ring
```

`field_simp [hchar]` multiplies by 2 and clears denominators, leaving
`λ + λ = 2*λ`. Then `ring` simplifies `λ+λ` to `2*λ`. Using `field_simp`
alone leaves `λ*(1+1) = 2*λ` which `field_simp` can't finish.

**Rule**: after `field_simp [hchar]`, always follow with `ring` (or `ring_nf`)
when the goal involves sums of the multiplied terms.

## `set` variable trap with external function hypotheses

When you `set X := coeffOf D i j i j` and then call an external lemma like
`eval_diagonal_invariant`, the returned hypothesis uses the RAW expression
`coeffOf D i j i j`, NOT the `set` abbreviation `X`. Subsequent `rw [hX]`
or `rw [hX] at h_conf` will fail silently.

```lean4
-- BAD: h_conf has raw coeffOf expressions, set vars can't match
set M1 := coeffOf D (i+1) (i+4) (i+1) (i+4) with hM1
have h_conf := eval_diagonal_invariant D i (i+4) (i+1) (i+2) ...
rw [hM1_expand, hM2_eq_M3] at h_conf  -- FAILS: h_conf has raw expressions

-- GOOD: use `simpa [hM1, hM2, hM3, ha_i]` to expand set vars before rewriting
-- OR: don't use `set` for variables that appear in externally-generated hypotheses
-- OR: use `← ha_i` to rewrite the raw expression TO the set var:
rw [← ha_i] at h_conf'  -- converts raw coeffOf to a_i
```

**When to use `set`**: only for local abbreviations where ALL references
(including from external function calls) are under your control.
For hypotheses coming from `eval_diagonal_invariant`, `eval_diagonal`,
or any lemma defined in a different file, the returned expressions use
the raw syntax — `set` abbreviations won't match.

**When NOT to use `set`**: when the abbreviated expression will appear
in externally-generated hypotheses that you can't control.
For `h_conf` from `eval_diagonal_invariant`, just use the raw
`coeffOf D ...` expressions directly.

When a typeclass carries a `(n : ℕ)` parameter and the function using it
doesn't mention `n` in its return type, Lean can't determine `n`, leaving
a metavariable:

```lean4
-- BAD (typeclass instance problem is stuck):
class ProjectionIdentity (F : Type) [Field F] (n : ℕ) where
  coeff : ℕ → ℕ → ℕ → ℕ → F

variable [ProjectionIdentity F n]
def coeffOf (i j u v : ℕ) : F := ...  -- n undetermined!

-- GOOD (explicit structure, n passed explicitly):
structure ProjectionIdentity (F : Type) [Field F] where
  n : ℕ
  coeff : ℕ → ℕ → ℕ → ℕ → F

def coeffOf (pi : ProjectionIdentity F) (i j u v : ℕ) : F := ...
  -- if hjn : j ≤ pi.n then ...  -- n accessed via pi.n
```

**Rule**: Use a `structure` (not `class`) when a parameter like `n` is needed
at runtime in if-guards. The structure makes `n` an explicit field accessible
via dot notation. A typeclass only works when `n` can be inferred from the
function's type signature.

## Namespace theorem ≠ structure field — use lambda wrapper

A `theorem` in a namespace is NOT a field of the structure, so `D.theorem_name`
fails. When filling a structure field that expects a function, wrap the
namespace theorem in an explicit lambda:

```lean4
-- BAD (Invalid field: not a structure field):
def toProjectionIdentity (D : NnDerivation F n) : ProjectionIdentity F where
  bracket_zero := D.bracket_zero  -- ERROR: bracket_zero is a theorem, not a field

-- GOOD (explicit lambda wrapping the theorem call):
def toProjectionIdentity (D : NnDerivation F n) : ProjectionIdentity F where
  bracket_zero := fun i j c d u v hi hij hjn hc hcd hdn hu huv hvn hj_ne_c hi_ne_d =>
    bracket_zero D i j c d u v hi hij hjn hc hcd hdn hu huv hvn hj_ne_c hi_ne_d
```

**Rule**: structure fields are `def`s/fields, namespace theorems are separate.
To convert a theorem into a field value, wrap it in a lambda that supplies the
first argument (`D`) explicitly.

---

## `simp` with `¬` on CONJUNCTION `if` conditions — use `split_ifs` instead

When `simp [h]` where `h : ¬ (a = b)` is applied to `if (a = b ∧ cond) then X else 0`,
`simp` may split the `if` into subgoals but fail to close the `a=b` branch
because the context has extra constraints (`1 ≤ k` preventing `k=0`) that
`simp` cannot see.

**Observed** (2026-06-26, `width_a_chain_zero` all-zero propagation):
```lean4
-- GOAL: (if (1:ℕ) = k+1 ∧ k+2 < n-1 then coeffOf ... else 0) = 0
-- With hk_ge1: 1 ≤ k (so k≠0, hence 1≠k+1)

-- BAD: simp [show ¬((1:ℕ)=k+1) from by omega] — leaves k=0 → ... goal
-- BAD: split_ifs with h <;> try rfl; exfalso; omega — rfl applies to wrong goal

-- GOOD: explicit split_ifs with named contradiction
have h1_ne_k1 : (1 : ℕ) ≠ k+1 := by omega
split_ifs with h
· exfalso; exact h1_ne_k1 h.1
· rfl
```

The `split_ifs with h` where `h` captures `(1=k+1 ∧ k+2<n-1)` gives
`h.1 : 1 = k+1`. The contradiction with the pre-proved `h1_ne_k1` closes
`simp` with `¬(P ∧ Q)` when simplifying `if P ∧ Q then ...`.

**Contrast with single-condition `if`**: `simp [h]` works fine for
`if a=b then X else 0` when `h : ¬ (a=b)`. The conjunction case is
different because `simp` treats `¬(P ∧ Q)` differently from `¬P` alone
when simplifying `if P ∧ Q then ...`.

---

## `simp` with equality-in-conjunction hypotheses over-rewrites — use `if_pos`/`if_neg`

When a hypothesis `hT2 : u = c ∧ d < v` contains a variable EQUALITY (here `u=c`),
`simp [hT2]` will use the equality to rewrite `u` to `c` EVERYWHERE in the goal,
including in coefficient expressions like `coeffOf D i j u c`.  This turns it
into `coeffOf D i j c c`, breaking any `coeffOf_f` lemma that was proved about
the original position `(i,j;u,c)`.

**Fix**: use `rw [if_pos hT2]` and `rw [if_neg hT2]` instead of `simp`:

```lean4
-- BAD (u=c equality rewrites coeffOf position):
simp [hT1, hT2, hT3, hT4, hT1_eq, hT2_eq, hT3_eq, hT4_eq]
-- → coeffOf i j u c becomes coeffOf i j c c, hT1_eq no longer matches

-- GOOD (if_pos/if_neg only touches the if-condition, not coeff expressions):
rw [if_pos hT1, hT1_eq, if_pos hT2, hT2_eq, if_pos hT3, hT3_eq, if_pos hT4, hT4_eq]
```

This is especially important for `bracketIdentity_eq_expanded` where 4 if-conditions
must be simplified.  In the `hT1` false branch, use `rw [if_neg hT1]`.

**Prefer `if_pos`/`if_neg` over `simp`** whenever any hypothesis contains a variable
equality (`x = y`) that could rewrite positions in coefficient expressions.
This was discovered during the coeffOf Phi bridge in D3 (2026-06-28).
