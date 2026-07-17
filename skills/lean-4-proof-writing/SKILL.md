---
name: lean-4-proof-writing
description: >-
  Guide for writing Lean 4 proofs — tactics reference, common proof patterns,
  and standard techniques for theorem proving. Designed for an AI agent to
  reference when helping a user write or verify Lean proofs.
  Covers: core tactics, automation tactics, calc/induction/cases patterns,
  mathlib tactics, decision tree, English-to-Lean phrasebook, proof templates.
  NOT about setting up the Lean environment (see lean-4-workflow).

Field pitfalls (linarith/simp/split_ifs/omega): see references/field-f-pitfalls.md.
---

# Lean 4 Proof Writing

## When to load

Load this skill whenever the user asks you to:
- Write a Lean 4 proof
- Debug a Lean proof error
- Choose the right tactic for a goal
- Understand a proof pattern (induction, calc, cases, etc.)
- Use mathlib tactics (ring, linarith, aesop, etc.)

## Pitfall: overwriting files with write_file

**NEVER use `write_file` to edit a Lean file unless you've just read the entire file
in the same turn.** `write_file` replaces the WHOLE file content. If you only intend
to modify one lemma, use `patch` with `old_string`/`new_string`. Only use `write_file`
for creating brand-new files from scratch. In this project, `write_file` was used twice
on `Bracket.lean` and both times destroyed working code (coeffOf, coeffOf_f, coeffOf_cond).

## Core rule: partial imports, not full Mathlib

**See also**: `references/bracket-case-analysis.md` — `split_ifs` + `‹...›` pattern, `induction'` with `generalizing` pitfall, and the `subst` vs `rw` issue after `split_ifs`.

After a full mathlib build, **do NOT `import Mathlib`** in .lean files.
Always import only the specific submodules needed. This affects both
compilation speed AND initial build time — `import Mathlib` in the project
entry point forces `lake build` to compile all ~8400 modules transitively:

```lean4
-- ❌ Slow (~60s per compile, hours for initial build): loads all ~8400 submodules
import Mathlib

-- ✅ Fast (~2s per compile, ~15-45 min initial build): loads only what's needed
import Mathlib.Data.Nat.Prime.Basic
import Mathlib.Data.Real.Basic
import Mathlib.Tactic
```

`lake build` compiles only the transitive closure of what the project's
entry point imports. Keeping the entry point's imports minimal directly
reduces the initial build from hours to minutes.

### ⚠️ Module paths change between mathlib versions

Many commonly referenced import paths from older tutorials **do not exist**
in the current mathlib (v4.31.0-rc1). They were renamed, merged into parent
modules, or restructured. **Do not assume a path exists** just because an old
Lean 3 / early Lean 4 tutorial uses it.

**Known-dead paths and their status (confirmed via mathlib4 GitHub source):**

| Dead import path | Status | Working alternative |
|---|---|---|
| `Data.Nat.Dvd.Basic` | No source file; dvd is a language primitive in `Init` | Just `Nat.dvd`, no extra import needed |
| `Data.Nat.Parity` | No source file; `Nat.even_or_odd` lives in `Nat.Basic`/`Std` | Import `Nat.Prime.Basic` or `Data.Real.Basic` |
| `Algebra.GroupPower.Basic` | No source file; power lemmas split across `Algebra/*/Power` | Import `Algebra.Group.Basic` or specific submodule |
| `LinearAlgebra.Basic` | No source file; split into submodules | Import `Algebra.Module.Basic` |
| `CategoryTheory.Basic` | No source file; split into submodules | Import specific submodule (e.g., `CategoryTheory.Core`) |
| `Topology.Instances.Real` | No source file | Import `Topology.Instances.Discrete` or a specific instance |
| `Algebra.Order.Basic` | No source file; split into `Algebra.Order.*` submodules | Import specific submodule |
| `Algebra.BigOperators.Basic` | No source file; split into `Algebra.BigOperators.*` | Import `Algebra.BigOperators.Finprod` or similar |
| `Analysis.SpecialFunctions.Trigonometric` | Was a file, became a **directory** of submodules | Use `Analysis.SpecialFunctions.Trigonometric.Basic` ✅ |

**Golden rule:** Before writing `import Mathlib.X.Y.Z`, verify the path exists:

```bash
# Method 1: Check source file exists
ls /opt/lean-home/lean-projects/e/.lake/packages/mathlib/Mathlib/X/Y/Z.lean

# Method 2: Check compiled .olean
ls /opt/lean-home/lean-projects/e/.lake/packages/mathlib/.lake/build/lib/lean/Mathlib/X/Y/Z.olean

# Method 3: Let lake tell you (fast)
echo 'import Mathlib.X.Y.Z' > /tmp/test.lean && \
  cd /opt/lean-home/lean-projects/e && source ~/.elan/env && \
  lake env lean /tmp/test.lean 2>&1 | grep error
```

See `references/verified-import-paths.md` for a comprehensive list of verified
working paths and their dead alternatives.

For a full automated check that all skill claims still match the installed
mathlib, run either verification script:

```bash
cd /opt/lean-home/lean-projects/e && source ~/.elan/env

# Quick build verification (12 import paths, 8 domains)
lake env lean E/Verify.lean

# Full skill audit (35 tactics, theorem names, patterns)
lake env lean E/SkillAudit.lean
```

Both must exit with zero errors on the current mathlib version.

## Tactic Decision Tree

When you don't know which tactic to use, follow this decision tree:

```
What's my goal?
├─ Close with exact term → exact / assumption
├─ Apply lemma to reduce goal → apply / refine
├─ Prove equality
│   ├─ By definition → rfl
│   ├─ By rewriting → rw [lemma]
│   ├─ By calculation → calc / ring / field_simp
│   └─ By extensionality → ext / funext
├─ Split goal
│   ├─ And/Iff → constructor
│   ├─ Or → left / right
│   └─ Exists → use witness
├─ Split hypothesis → cases / rcases / obtain
├─ Simplify → simp / norm_num / ring
└─ Don't know → exact? / apply? / simp?
```

## Automation Tactics Cascade

When stuck, try tactics in this order (stop on first success):

`rfl` → `simp` → `ring` → `linarith` → `nlinarith` → `omega` → `grind` → `aesop`

## Quick Reference Table

| Want to... | Use... |
|------------|--------|
| Close with exact term | `exact` |
| Apply lemma | `apply` |
| Rewrite once | `rw [lemma]` |
| Normalize expression | `simp`, `ring`, `norm_num` |
| Split cases | `by_cases`, `cases`, `rcases` |
| Prove exists | `use witness` |
| Prove and/iff | `constructor` |
| Prove function equality | `ext` / `funext` |
| Explore options | `exact?`, `apply?`, `simp?` |
| Automate domain-specific | `ring`, `linarith`, `continuity`, `measurability` |
| Cross-domain automation | `grind` (SMT-style) |
| Linear arithmetic (ℕ/ℤ) | `omega` |
| Decidable propositions | `decide` / `native_decide` |
| Proof search | `aesop` |
| Chain relations | `calc` |
| Proof by contradiction | `by_contra` |
| Case split on proposition | `by_cases` |
| Introduce hypothesis | `intro` |
| Intermediate lemma | `have` |

## Tactic reference (all 35 tactics, 5 categories)

### Essential Tactics Explained

#### `simp` — The Workhorse Simplifier

Recursively applies `@[simp]` lemmas to rewrite expressions to normal form.

```lean4
simp                    -- Use all simp lemmas
simp only [lem1, lem2]  -- Use only specified lemmas (preferred)
simp [h]                -- Use hypothesis h as a simp lemma
simp at h               -- Simplify hypothesis h
simp at *               -- Simplify all hypotheses and goal
simpa using h           -- simp then exact h
simp?                   -- Show which lemmas it uses (exploration)
```

**When to use `simp`:** Obvious algebraic simplifications (`x + 0 = x`, `x * 1 = x`, etc.),
normalizing expressions, cleaning up after other tactics.

**When to use `simp only`:** You know which lemmas you need — preferred for
explicit, reviewable proofs.

#### `rw` — Rewriting

```lean4
rw [h]              -- rewrite goal using h
rw [h] at h1        -- rewrite h1 using h
rw [h1, h2]         -- chain rewrites
rw [h] at *         -- rewrite everywhere
rw [← h]            -- rewrite backwards (h reversed)
rw [add_comm a b]   -- rewrite with a specific lemma
```

#### `calc` — Chain Relations

```lean4
example (a b c d : ℕ) (h1 : a = b) (h2 : b = c) (h3 : c = d) : a = d := by
  calc
    a = b := h1
    _ = c := h2
    _ = d := h3

-- Also works with ≤, ≥, <, >, etc.
example (a b : ℝ) (h : a ≤ b) : a ≤ b + 1 := by
  calc
    a ≤ b := h
    _ ≤ b + 1 := by nlinarith
```

**Calc chain critical pattern:** After `simp`, always check the actual goal state
BEFORE starting the calc chain. `simp` may or may not transform the goal
depending on context. Start the calc from whatever form the goal is actually in.

**Calc block pitfalls (common source of "No goals to be solved" errors):**
- **`rw` can close a calc subgoal too eagerly.** If `rw [h]` rewrites the goal into `rfl`-form, the subgoal is closed immediately, and any subsequent `simp`/`ring_nf` at the same step produces "No goals to be solved". Fix: separate the rewrite into its own `calc` step, or use an intermediate `have` lemma.
- **`calc` block CAN reference `h.2` from a parent `dite`/`if h : ...`** — when a `calc` step directly quotes an expression `(if h : P then ... h.2 ... else 0)`, the binder `h` is valid inside the `calc` term because the `if` creates a fresh binder. This is useful when using a `h_cond` that contains `dite` with `h` binder in a `calc` block:
  ```lean4
  have h_cond := D.half_deriv_cond x y k hk hk'
  dsimp [x, y, MatIdx.left, MatIdx.right] at h_cond
  -- h_cond: (2:F)*coeff x y = (if h : v=j ∧ u<k then coeff ⟨i,k⟩ (⟨u,k,hu,h.2,...⟩) else 0) - ...
  calc
    (2:F)*coeff x y = (if h : v=j ∧ u<k then coeff ... (⟨u,k,hu,h.2,...⟩) else 0) - ... := h_cond
    _ = ... := ...
  ```
  The `h.2` inside the first `calc` step is valid — the `(if h : ...)` expression creates a fresh binder.
- **Nested `calc` blocks (calc inside `:=` of another calc) cause errors.** Instead, break the outer calc into separate `have` lemmas and chain them:
  ```lean4
  -- ❌ Bad: nested calc
  have h : A = D := by
    calc
      A = B := by
        calc A = X := h1; _ = B := h2  -- nested calc → error
      _ = C := h3
      _ = D := h4

  -- ✅ Good: break into intermediate lemmas
  have h_AB : A = B := by calc A = X := h1; _ = B := h2
  have h : A = D := by calc A = B := h_AB; _ = C := h3; _ = D := h4
  ```
- **Long calc chains (>3 steps) are hard to debug.** Prefer breaking into separate `have` lemmas with descriptive names. Each lemma should prove one conceptual step. Then chain them in a short calc.
- **`calc` with `ring_nf` + `rw` in the same `:=` step** often causes "No goals to be solved". Apply them in separate calc steps.
- **`calc` result doesn't match goal after `subst`:** After `subst i` (where `i : ℕ`), the goal may have `(n-1)+1` instead of `n`. The `calc` block still computes the unsimplified form. Fix: wrap in a `have` and `simpa` with the simplification lemma: `simpa [show (n-1 : ℕ) + 1 = n by omega] using h_result`.

**Proving structural equality by linear combination (`calc` + `rw`):** When defining
`T = D - c·Id` inside an anonymous `HalfDerivation` constructor, the field
`half_deriv_cond` must be proved by combining `D.half_deriv_cond` and
`Id.half_deriv_cond`. The key technique: define the coefficient as a standalone `def`,
then use `calc` + `rw` to show RHS equality by linearity:

```lean4
noncomputable def T_coeff (D : HalfDerivation F n) := λ x y => D.coeff x y - c * Id.coeff x y

noncomputable def T (D : HalfDerivation F n) : HalfDerivation F n :=
  { coeff := T_coeff D
    half_deriv_cond := by
      intro x y k hk hk'
      have hD := D.half_deriv_cond x y k hk hk'
      have hId := Id.half_deriv_cond x y k hk hk'
      have h_if_distrib (cnd : Prop) [Decidable cnd] (a b : F) :
          (if cnd then a - c * b else (0 : F)) = (if cnd then a else (0 : F)) - c * (if cnd then b else (0 : F)) := by
        split <;> simp
      dsimp [T_coeff]
      calc
        (2 : F) * (D.coeff x y - c * Id.coeff x y)
            = (2 : F) * D.coeff x y - c * ((2 : F) * Id.coeff x y) := by ring
        _ = ((2 : F) * D.coeff x y - c * ((2 : F) * Id.coeff x y)) := rfl
        _ = (((2 : F) * D.coeff x y) - c * ((2 : F) * Id.coeff x y)) := rfl
        _ = ((2 : F) * D.coeff x y) - c * ((2 : F) * Id.coeff x y) := rfl
        _ = ((2 : F) * D.coeff x y - c * ((2 : F) * Id.coeff x y)) := rfl
      -- TODO: complete the chain by rw [hD, hId]; simp [h_if_distrib, ...]; ring
  }
```

Then fill the `calc` chain to reach the RHS:

```lean4
      calc
        (2 : F) * (D.coeff x y - c * Id.coeff x y)
            = (2 : F) * D.coeff x y - c * ((2 : F) * Id.coeff x y) := by ring
        _ = ((2 : F) * D.coeff x y) - c * ((2 : F) * Id.coeff x y) := rfl
        _ = ((RHS_D) ...) - c * ((RHS_Id) ...) := by
          rw [hD, hId]
        _ = ((if ... then D.coeff ... - c * Id.coeff ... else 0) - ... + ...) := by
          simp [T_coeff, h_if_distrib, add_comm, add_left_comm, add_assoc,
            sub_eq_add_neg, mul_add, smul_eq_mul, mul_comm, mul_left_comm, mul_assoc]
          ring
```

The `rw [hD, hId]` step replaces `2*D.coeff x y` with `RHS_D` and `2*Id.coeff x y`
with `RHS_Id`. The follow-up `simp` uses `h_if_distrib` to rewrite each
`if cnd then (D.coeff ... - c*Id.coeff ...) else 0` into
`(if cnd then D.coeff ... else 0) - c*(if cnd then Id.coeff ... else 0)`,
which matches the `RHS_D - c*RHS_Id` form. `ring` handles the algebraic factoring.

**Note:** This `calc` chain is the general approach to proving linearity of
`half_deriv_cond`. The technique works for ANY structure with field conditions
that are linear in the structure's data.

✅ Pattern for long chains:
```lean4
-- Build step-by-step: each intermediate lemma is one calc step
have ha1 : A = B := by calc ...  -- 1-2 steps
have ha2 : B = C := by calc ...  -- 1-2 steps  
have ha3 : C = D := by
  calc
    C = C1 := by rw [h1]     -- single rewrite, separate step
    _ = D := by ring_nf       -- normalization, separate step
have h_result : A = D := by calc
  A = B := ha1
  _ = C := ha2
  _ = D := ha3
```

### Core tactics (9)

| Tactic | Purpose | Example |
|--------|---------|---------|
| `rfl` | 证明等式两边计算上相等（自反性） | `by rfl` |
| `exact h` | 用现有的假设/定理精确匹配目标 | `exact h` 当目标就是 `h` 时 |
| `apply h` | 应用一个函数/定理来转化目标 | `apply h` 当 `h : A → B` 且目标是 `B` |
| `intro h` | 引入蕴含/∀ 的前提作为假设 | `intro h` 将 `A → B` 的目标转化为假设 `h : A` 下证 `B` |
| `rw [h]` | 用等式重写目标或假设 | `rw [add_comm]` 交换加法顺序 |
| `simp` | 用 `@[simp]` 引理库化简 | `simp` 自动化简 |
| `omega` can't prove `i+1 < j` from `j-i > 1` | `omega` may struggle when the relationship is buried in subtraction | Provide explicit bound: `have hmid : i+1 < j := by have : 1 < j-i := hdgt; omega` |
| `No goals to be solved` after `omega; omega` | `omega` on the first branch closes the goal, second `omega` has nothing to do | Use `omega` once: `by omega` not `by omega; omega`. Use separate `have` lines for intermediate facts, then a single `omega` |
| `noncomputable def` won't reduce with `rfl`/`dsimp`/`simp` | Lean 4 compiles `noncomputable def` as opaque — definitional reduction disabled | Use `unfold` (works when def is in scope) or `rw [myDef]` (uses equation lemma). `rfl` NEVER works on noncomputable defs |
| `≠` direction mismatch: `x.i ≠ i` vs `¬(i = x.i)` | `≠` is `¬ (a = b)` with a specific argument order; binder names in theorem signatures generate different `≠` orientations | Use `Ne.symm h` to flip direction, or `rcases` with `h` and `h.symm` introduced separately |
| `variable (F n)` makes F,n explicit → `T D` fails | Explicit binder requires `T F n D`; Lean reports "Application type mismatch" or treats `D` as Type arg | Add F,n: `T F n D`. Use `variable {F n}` with braces to make implicit if desired |
|--------|---------|---------|
| `omega` | 解 ℕ/ℤ 上的线性算术 | `omega` |
| `decide` | 判定可判定的命题（有限搜索） | `decide` |
| `native_decide` | 同 decide 但用本地编译，更快 | `native_decide` |
| `aesop` | 自动化证明搜索（多策略组合） | `aesop` |
| `simp [h1, h2]` | 用指定引理化简 | `simp [add_comm, add_left_comm]` |
| `simp_all` | 化简目标和所有假设 | `simp_all` |

### Advanced tactics (7)

| Tactic | Purpose | Example |
|--------|---------|---------|
| `calc` | 链式等式/关系推导 | `calc a = b := h1; _ = c := h2` |
| `conv` | 深入到子表达式中做重写 | `conv => lhs; rw [h]` |
| `by_contra h` | 反证法 | `by_contra h` 引入 `h : ¬目标` |
| `by_cases h : P` | 对命题分情况 | `by_cases h : x = 0` |
| `<;>` | 对所有剩余目标执行 tactic | `constructor <;> simp` |
| `first` | 依次尝试，用第一个成功的 | `first | apply h | apply h2` |
| `repeat` | 重复直到失败 | `repeat rw [h]` |

### Mathlib tactics (8)

| Tactic | Purpose | Requirements |
|--------|---------|-------------|
| `ring` | 证明多项式等式 | `import Mathlib.Tactic` |
| `field_simp` | 分式化简去分母 | `import Mathlib.Tactic` |
| `linarith` | 线性不等式 | `import Mathlib.Tactic` |
| `nlinarith` | 非线性算术 | `import Mathlib.Tactic` |
| `polyrith` | 用 Gröbner 基找多项式证明 | `import Mathlib.Tactic` |
| `rcases h with ...` | 递归解构假设 | (自带) |
| `obtain h := ...` | 解构并引入存在量词 | (自带) |
| `push_neg` | 用德摩根律推进否定（**已废弃**，改用 `push Not`） | `import Mathlib.Tactic` |

`push Not` works for simple negations. For complex nested `¬(A ∨ B ∨ C ∨ D)` where each clause is a multi-part `∧`, `push Not` may stall with "made no progress". **Reliable replacement:**

```lean4
h_not : ¬((P ∧ Q ∧ R) ∨ (S ∧ T ∧ U) ∨ (V ∧ W ∧ X) ∨ (Y ∧ Z ∧ A))

-- ❌ push Not at h_not  →  "made no progress"

-- ✅ simpa + not_or, not_and_or
have h_not' : ¬(P ∧ Q ∧ R) ∧ ¬(S ∧ T ∧ U) ∧ ¬(V ∧ W ∧ X) ∧ ¬(Y ∧ Z ∧ A) := by
  simpa [not_or, not_and_or] using h_not
rcases h_not' with ⟨h1, h2, h3, h4⟩
-- each hX is now ¬(P ∧ Q ∧ R), i.e. ¬P ∨ ¬Q ∨ ¬R
```

Also: `Ne.def` does NOT exist in Lean 4. If you need the `Ne` equation lemma, use `ne_eq` instead.

### Project-specific lemma: `CharNeTwo.char_ne_two`

The custom `CharNeTwo` class (defined in `E/HalfDerivation/Infrastructure.lean`) provides:
```lean4
class CharNeTwo (F : Type) [Field F] where
  char_ne_two : (2 : F) ≠ 0
attribute [simp] CharNeTwo.char_ne_two
```
Access it as `CharNeTwo.char_ne_two` (not `CharNeTwo.two_ne_zero` — that lemma does not exist). It's `@[simp]` so `simp` can use it automatically when the instance is available.

## Proof Templates

> **Note on templates and mathlib style**: The templates below use `--` comments
> for strategy guidance. These are **development scaffolding** — use them while
> writing. Before submitting to mathlib, strip all inline comments per the
> [Mathlib Code Style](#mathlib-code-style) rules below. The templates serve as
> a thinking framework, not final formatting.

### General Theorem Template

```lean4
theorem my_theorem (n : ℕ) : conclusion := by
  -- Strategy: Describe proof approach here
  -- Step 1: [Describe what needs to be shown]
  have h1 : _ := by
    sorry

  -- Step 2: [Describe next step]
  have h2 : _ := by
    sorry

  -- Step 3: Combine results
  sorry
```

### Induction Template

```lean4
theorem induction_example (n : ℕ) : P n := by
  induction n with
  | zero =>
      -- Base case: n = 0
      sorry
  | succ n ih =>
      -- Inductive step: assume P(n), prove P(n+1)
      -- Inductive hypothesis: ih : P(n)
      sorry
```

### Strong induction gotcha with `generalizing`

When using `induction' ... using Nat.strong_induction_on with d ih generalizing i j`:

- The induction hypothesis `ih` captures **all** hypotheses about `i` and `j` that were in the context before `induction'`. This includes `hlen : j - i = d`, `hpos : 0 < j - i`, etc.
- The IH type becomes: `∀ m < d, ∀ (i' j' : ℕ), 1 ≤ i' → i' < j' → j' ≤ n → 0 < j' - i' → (j' - i' = m) → sys.C i' j' = sys.a 1`
- You MUST provide all these hypotheses when calling `ih`. The `rfl` at the end satisfies `(j' - i' = m)` when `m = j' - i'`.

```lean4
-- Example:
induction' hlen : j - i using Nat.strong_induction_on with d ih generalizing i j
-- ih: ∀ m < d, ∀ (i' j' : ℕ), 1 ≤ i' → i' < j' → j' ≤ n → 0 < j' - i' → (j' - i' = m) → sys.C i' j' = sys.a 1

-- Calling for C(i,k) where k-i < j-i:
have h_ik : sys.C i k = sys.a 1 :=
  ih (k - i) (by omega) i k hi (by omega) (by omega)
    (Nat.sub_pos_of_lt hik) rfl
```

### ⚠️ `generalizing` captures `by_cases` hypotheses too

When `generalizing` is active AND a `by_cases` introduces a hypothesis BEFORE `induction'`, the hypothesis gets captured by `generalizing`:

```lean4
-- ❌ h_ge2 is captured by generalizing, REQUIRED in every ih call
lemma nonadjacent (D : NCoeff F n) (… ) (h_ge2 : j - i ≥ 2) : D.f i j u v = 0 := by
  induction' hlen : v - u using Nat.strong_induction_on with d ih generalizing i j u v
  -- ih REQUIRES (j' - i' ≥ 2) for EVERY sub-call
  -- When j-i = 2, k = i+1 gives k-i = 1 < 2 → can't prove → STUCK
```

Three ways to avoid this:

**Option A** (cleanest): put `by_cases h_ge2` INSIDE the induction block, after `induction'`:
```lean4
lemma nonadjacent (D : NCoeff F n) (…) : D.f i j u v = 0 := by
  induction' hlen : v - u using Nat.strong_induction_on with d ih generalizing i j u v
  by_cases h_ge2 : j - i ≥ 2
  · by_contra! h_nonzero
    …  -- ih does NOT require h_ge2, works for all cross-terms
  · …  -- handle adjacent case (j = i+1) separately
```

**Option B** (explicit `revert`): push `h_ge2` out before `induction'` then bring it back:

```lean4
lemma nonadjacent (D : NCoeff F n) (…) (h_ge2 : j - i ≥ 2) : D.f i j u v = 0 := by
  revert h_ge2
  induction' hlen : v - u using Nat.strong_induction_on with d ih generalizing i j u v
  intro h_ge2
  -- ih type: ∀ d'<d, ∀ i' j' u' v', v'-u' = d' → ((j'-i' ≥ 2) → …)
  -- Calling ih requires proving (j'-i' ≥ 2) as a premise.
  -- Same problem as before when j-i=2, k=i+1: need k-i ≥ 2 which is false.
```

**Option C** (inner `have`, best for reuse): write `h_nonadjacent` as a `have` block with its own `induction'`, BEFORE the main proof:

```lean4
theorem diagonalization (D : NCoeff F n) (h2ne : (2 : F) ≠ 0) : … := by
  have h_nonadjacent : ∀ i j u v, … → j - i ≥ 2 → D.f i j u v = 0 := by
    intro i j u v hi hij hjn h_ge2 hu huv hvn h_off
    induction' hlen : v - u using Nat.strong_induction_on with d ih generalizing i j u v
    -- ih does NOT require h_ge2 (it's inside the have, outside induction')
    …
  intro i j u v … h_off
  by_cases h_ge2 : j - i ≥ 2
  · exact h_nonadjacent i j u v … h_off h_ge2
  · …  -- adjacent case uses h_nonadjacent for cross-terms
```

**Why Option A wins:** The `ih` covers ALL target-length-shorter pairs regardless of source non-adjacency (`j'-i' ≥ 2`). This is critical because `source_exists` creates cross-terms with source `(i,i+1)` (adjacent, length 1) when the original source is `(i,i+2)` (non-adjacent, length 2). The IH must apply to those adjacent-source subproblems to get a contradiction.

**Why Option B is still stuck:** After `revert h_ge2` + `generalizing`, the IH turns `(j'-i' ≥ 2)` into a PREMISE (`((j'-i' ≥ 2) → …)`), not a condition. You CAN call `ih` for `(i,i+1)` — you just can't prove `(i+1)-i ≥ 2`. Same dead end as having `h_ge2` in the parameter list.

```lean4
-- TYPES COMPARISON:
-- h_ge2 as PARAMETER:   ih : ∀ d'<d, ∀ i' j' u' v', … → (j'-i' ≥ 2) → … → result
-- revert h_ge2:         ih : ∀ d'<d, ∀ i' j' u' v', v'-u'=d' → ((j'-i' ≥ 2) → …)
-- by_cases h_ge2 INSIDE: ih : ∀ d'<d, ∀ i' j' u' v', v'-u'=d' → … → result
```

```lean4
theorem cases_example (h : a ∨ b) : c := by
  cases h with
  | inl h_left =>
      -- Case 1: Left branch, available: h_left
      sorry
  | inr h_right =>
      -- Case 2: Right branch, available: h_right
      sorry
```

### Calculation Chain Template

```lean4
theorem calc_example : a = d := by
  calc a = b := by
      -- TODO: Prove a = b
    _ = c := by
      -- TODO: Prove b = c
    _ = d := by
      -- TODO: Prove c = d
```

### Existential Proof Template

```lean4
theorem exists_example : ∃ x, P x := by
  use witness  -- provide the witness
  -- prove P(witness)
```

## English-to-Lean Phrasebook

### Forward Reasoning (从前提出发推导)

| English | Lean |
|---------|------|
| "Observe that A holds because of reason r" | `have h : A := by r` |
| "From hypothesis h, applying f gives us B" | `have hB : B := f h` |
| "By definition, X = Y" | `have : X = Y := rfl` |

### Backward Reasoning (从目标向后推导)

| English | Lean |
|---------|------|
| "It suffices to show A" | `suffices hA : A by ...` |
| "By theorem T, we need to show A" | `apply T` |
| "We prove by induction on n" | `induction n with ...` |

### Case Analysis

| English | Lean |
|---------|------|
| "Consider two cases: P or not P" | `by_cases h : P` |
| "Case 1: P holds" | first branch after `by_cases` |
| "Case 2: ¬P holds" | second branch |

### Rewriting and Simplification

| English | Lean |
|---------|------|
| "Replace X with Y using lemma h" | `rw [h]` |
| "This simplifies to ..." | `simp` |
| "After simplification, the goal becomes ..." | `simp at *` |

### Connectives (逻辑连接词)

| English | Lean |
|---------|------|
| "We need to prove both A and B" | `constructor` |
| "We need to prove A or B" — choose left | `left` |
| "We need to prove A or B" — choose right | `right` |
| "If A, then B" | `intro hA` |
| "A if and only if B" | `constructor` (two goals) |

### Quantifiers (量词)

| English | Lean |
|---------|------|
| "For all x, P(x)" → introduce x | `intro x` |
| "There exists x such that P(x)" → provide witness w | `use w` |
| "Given ∃ x, P(x)", extract x and evidence | `rcases h with ⟨x, hx⟩` |

### Contradiction

| English | Lean |
|---------|------|
| "Suppose for contradiction that ¬A" | `by_contra h` |
| "This contradicts h" | `exact h hn` |
| "Therefore A" (after contradiction) | `exact absurd h' h` |

### Inequalities

| English | Lean |
|---------|------|
| "It suffices to show a ≤ b" | `apply le_of_lt` etc. |
| "By transitivity, a ≤ c" | `calc a ≤ b := h1; _ ≤ c := h2` |
| "From h1 and h2, we get a ≤ c" | `linarith` |

## Common patterns with core tactics

### Implication / ∀:
```lean4
example (h : ∀ x, P x) (a : A) : P a := by
  exact h a

example (h : A → B) (ha : A) : B := by
  apply h
  exact ha
```

### Conjunction (∧):
```lean4
example (ha : A) (hb : B) : A ∧ B := by
  exact And.intro ha hb
  -- or: constructor; exact ha; exact hb

example (h : A ∧ B) : A := by
  exact h.left
  -- or: rcases h with ⟨ha, hb⟩; exact ha
```

### Disjunction (∨):
```lean4
example (ha : A) : A ∨ B := by
  left; exact ha

example (hb : B) : A ∨ B := by
  right; exact hb
```

### Negation (¬):
```lean4
example (h : A) (hn : ¬ A) : False := by
  exact hn h

example (h : ¬ A) (ha : A) : B := by
  exact absurd ha h
```

### Existential (∃):
```lean4
example (a : A) (h : P a) : ∃ x, P x := by
  exact ⟨a, h⟩

example (h : ∃ x, P x) : R := by
  rcases h with ⟨x, hx⟩
  -- now use x and hx
```

### Equality (=):
```lean4
example : a + b = a + b := by rfl

example (h : a = b) : a + c = b + c := by
  rw [h]

example (h : a = b) (h2 : b = c) : a = c := by
  calc
    a = b := h
    _ = c := h2
```

## Using mathlib theorems

Most mathlib theorems follow naming conventions:

```lean4
add_comm a b         -- a + b = b + a
add_assoc a b c      -- (a + b) + c = a + (b + c)
mul_comm a b         -- a * b = b * a
mul_assoc a b c      -- (a * b) * c = a * (b * c)
add_comm 0 a         -- 0 + a = a + 0 (= a)
sub_add_cancel a b   -- (a - b) + b = a (if b ≤ a)
Nat.succ_eq_add_one  -- Nat.succ n = n + 1
```

Use `#check` to find theorems:
```lean4
#check add_comm
#check Nat.add_comm
#check Real.sin_sq_add_cos_sq
```

## Verification Ladder

当编译 Lean 代码时，使用最轻量的验证方式：

```
Tier 1: 逐行检查       lean_diagnostic_messages(file)     亚秒级
Tier 2: 单文件编译      lake env lean <path/to/File.lean>   秒级
Tier 3: 全项目构建      lake build                          分级
```

**规则：**
- 每次编辑后：Tier 1（逐行检查）
- 文件级验证：Tier 2（单文件编译，在项目根目录执行，传相对路径）
- 检查点/最终验证：Tier 3（全项目构建）
- **不要**直接用 `lake build <文件名>`——lake 不支持文件参数

## LSP-First Protocol（MCP 工具优先）

如果配置了 lean-lsp-mcp MCP 服务器，优先使用 MCP 工具而非编译命令。
MCP 工具提供亚秒级反馈，替代 `lake env lean` 的秒级编译周期。

### ⚠️ MCP 初始化需要 mathlib 全量编译完成

`lean-lsp-mcp` 在启动时依赖所有 `.olean` 文件就绪。如果 mathlib 尚未全量编译
（或 `lake clean` 清空了缓存），`hermes mcp test lean_lsp` 会**超时**（30s+），
且 MCP 工具（`lean_goal`, `lean_diagnostic_messages` 等）无法正确响应。

**修复：** 确保 `lake build` 已完成（8496 jobs 零错误）。
之后重启 Hermes 或重新连接 MCP 即可。

**验证 MCP 是否就绪的方法：**
```bash
# 方式 1：hermes mcp test（需 mathlib 已 build）
hermes mcp test lean_lsp

# 方式 2：直接通过 JSON-RPC 测试（mathlib 未 build 时也能做基础测试）
echo '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"test","version":"1.0"}}}
{"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}' | \
  cd /opt/lean-home/lean-projects/e && LEAN_PROJECT_PATH=$PWD uvx lean-lsp-mcp
```

### Proof 阶段流程

**Planning（发现 sorry 阶段）：**
1. `mcp_lean_lsp_lean_goal(file, line)` — 理解目标
2. 最多 3 个搜索工具（时间盒 ~30s）：先 `lean_local_search`，其次 `lean_leanfinder`/`lean_leansearch`/`lean_hammer_premise`
3. 记录候选引理和下一步策略

**Work（填充阶段）：**
1. 开始时刷新 `lean_goal(file, line)`
2. 从搜索结果生成 2-3 个候选证明片段
3. 用 `lean_multi_attempt(file, line, snippets=[...])` 并行测试
4. `lean_diagnostic_messages(file)` 验证
5. 如果有 "Try this" 建议 → `lean_code_actions(file, line)` → 应用 → 重新验证

### Stuck 检测

当连续 2-3 次对同一个 sorry 尝试新方法都失败时，不要继续硬试。
应该：重新分析目标 → 搜索更多引理 → 生成新计划 → 尝试。

## 4-Phase Workflow

写 Lean 证明的四阶段流程：

```
Phase 1: Structure Before Solving  — 先列骨架，用 have + sorry 搭框架
Phase 2: Helper Lemmas First       — 提取可复用的辅助引理
Phase 3: Incremental Filling       — 每次填一个 sorry，编译，提交
Phase 4: Type Class Management     — 用 haveI/letI 处理实例合成
```

### Phase 1: Structure Before Solving

在写具体 tactic 之前，先用 `have` 和 `sorry` 搭出证明结构：

```lean4
theorem my_theorem (n : ℕ) : conclusion := by
  have h1 : intermediate_lemma_1 := by
    sorry  -- 稍后填充
  have h2 : intermediate_lemma_2 := by
    sorry  -- 稍后填充
  -- 然后组合 h1 和 h2 得到结论
  sorry
```

这让证明策略先于实现细节。

### Phase 2: Helper Lemmas First

先提取可复用的辅助引理，再证明主定理：

```lean4
lemma helper (x : ℕ) : property x := by
  sorry  -- 先在辅助引理上工作

theorem main : conclusion := by
  apply helper
```

### Phase 3: Incremental Filling

每次只填一个 `sorry`，编译通过后再填下一个：

```lean4
-- Step 1: 填第一个 sorry
theorem step1 : A → B := by
  exact λ h => h

-- Step 2: 编译通过，再填下一个
theorem step2 : B → C := by
  sorry  -- 现在在这工作
```

### Phase 4: Type Class Management

当遇到 `failed to synthesize instance` 时：

```lean4
-- 用 haveI 添加局部的类型类实例
haveI : MeasurableSpace Ω := inferInstance

-- 用 letI 提供显式实例
letI : Fintype α := ⟨...⟩

-- 用 letI 代替 haveI 当实例需要参数时
letI : DecidableEq α := λ a b => ...
```

顺序重要：先提供外层结构，再提供内层结构。

## One Step at a Time

**写一个 tactic，检查诊断，再写下一个。不要一次写多个 tactic 再编译。**

```lean4
-- ❌ 错误：一次写多个 tactic
example (P Q : Prop) (hP : P) (hQ : Q) : P ∧ Q := by
  constructor
  exact hP
  exact hQ  -- ← 如果 constructor 失败了，这行也是错的

-- ✅ 正确：逐个 tactic，每步验证
example (P Q : Prop) (hP : P) (hQ : Q) : P ∧ Q := by
  constructor        -- 第一步：检查目标变成 P 和 Q 两个子目标
  · exact hP         -- 第二步：填第一个子目标
  · exact hQ         -- 第三步：填第二个子目标
```

- 用 `done` 标记你需要继续的地方
- `by sorry` 可以用于当前不主动工作的占位符

## Bridging HalfDerivation to NCoeff for nonadjacent vanishing

A common pattern: prove that T(E(i,j)) vanishes for all targets when `j-i ≥ 2`
(nonadjacent source). This uses the NCoeff bridge but only `cond`, not `cond_zero`.

### Pattern: `T_nonadjacent_zero`

```lean4
theorem T_nonadjacent_zero (D : HalfDerivation F n) (hn5 : 5 ≤ n) (h2ne : (2 : F) ≠ 0)
    (x y : MatIdx n) (hlen : x.j - x.i ≥ 2) : (T F n D).coeff x y = 0 := by
  have hi : 1 ≤ x.i := x.hpos
  have hij : x.i < x.j := x.hlt
  have hjn : x.j ≤ n := x.hle
  have hu : 1 ≤ y.i := y.hpos
  have huv : y.i < y.j := y.hlt
  have hvn : y.j ≤ n := y.hle
  -- Diagonal case: use T_derived_zero
  by_cases h_diag : x = y
  · subst h_diag; exact T_derived_zero F n D hn5 x hlen
  · -- Off-diagonal: use NCoeff.diagonalization on toNCoeff D
    -- ⚠️ diagonalization only needs `cond`, not `cond_zero`
    have h_off' : y.i ≠ x.i ∨ y.j ≠ x.j := by
      by_contra! h; rcases h with ⟨hi_eq, hj_eq⟩
      apply h_diag; exact (MatIdx.ext_iff x y).mpr ⟨hi_eq, hj_eq⟩
    have h_off_swapped : x.i ≠ y.i ∨ x.j ≠ y.j := by
      rcases h_off' with (h | h); · left; exact Ne.symm h; · right; exact Ne.symm h
    have h_nc_all := NCoeff.diagonalization (HalfDerivation.toNCoeff D) h2ne
    have h_f_zero : (HalfDerivation.toNCoeff D).f x.i x.j y.i y.j = 0 :=
      h_nc_all x.i x.j y.i y.j hi hij hjn hu huv hvn h_off' hlen
    dsimp [HalfDerivation.toNCoeff] at h_f_zero
    -- Now: HalfDerivation.coeffOf D x.i x.j y.i y.j = 0
    have h_Dcoeff : D.coeff x y = 0 := by
      rw [← HalfDerivation.coeffOf_f D x.i x.j y.i y.j hi hij hjn hu huv hvn, h_f_zero]
    have h_Idcoeff : ((idHalfDeriv F n).coeff x y) = 0 :=
      id_off_diag F n x y h_off_swapped
    rw [T_coeff_eq F n D x y, h_Dcoeff, h_Idcoeff]
    ring
```

### Key steps

1. **Split diagonal/off-diagonal**: Diagonal uses `T_derived_zero` (already proven). Off-diagonal uses the NCoeff bridge.
2. **`NCoeff.diagonalization` only needs `cond`** — it does NOT depend on `cond_zero`. This means the bridge is usable even when `coeffOf_cond_zero` is still `sorry`.
3. **Convert `≠` directions**: `NCoeff.diagonalization` expects `y.i ≠ x.i ∨ y.j ≠ x.j` but `id_off_diag` expects `x.i ≠ y.i ∨ x.j ≠ y.j`. Keep both versions — don't try to unify them.
4. **`dsimp [HalfDerivation.toNCoeff]`** expands `(toNCoeff D).f` to `coeffOf D`. This works because `toNCoeff` is a `noncomputable def` with a `where` block (not a `let` binder).
5. **`coeffOf_f` reverses**: `coeffOf D i j u v = D.coeff ⟨i,j,...⟩ ⟨u,v,...⟩`. The goal has `D.coeff x y` where `x = ⟨i,j,...⟩`. Use `rw [← coeffOf_f ...]` to match.

### Why this works without cond_zero

`NCoeff.diagonalization` only calls `D.cond` (the half-derivation splitting rule) and `source_exists` (which also only uses `cond`). It never touches `cond_zero`. So `HalfDerivation.toNCoeff D` with `cond_zero := sorry` is a valid input — the `sorry` is opaque and the proof paths that matter never inspect it.

This is the key insight that lets us prove `T([L,L]) = 0` (all targets) BEFORE completing the bracket expansion bridge.



A recurring proof pattern in combinatorial Lie algebra formalization:
prove that certain coefficients are zero by strong induction on target
length, using a decomposition lemma (`source_exists`) that identifies
source terms with strictly shorter target lengths.

### Pattern

```lean4
theorem diagonalization (D : NCoeff F n) (h2ne : (2 : F) ≠ 0) : … := by
  intro i j u v hi hij hjn hu huv hvn h_off
  induction' hlen : v - u using Nat.strong_induction_on with d ih
    generalizing i j u v
  by_contra! h_nonzero
  …
```

The induction hypothesis `ih` has type:
```lean4
ih : ∀ (d' : ℕ), d' < d → ∀ (i' j' u' v' : ℕ), 1 ≤ i' → i' < j' → j' ≤ n →
      1 ≤ u' → u' < v' → v' ≤ n → (u' ≠ i' ∨ v' ≠ j') → D.f i' j' u' v' = 0
```

### ⚠️ Critical trap: source_exists requires `i < k < j`

If you use a lemma `source_exists` that takes `k : ℕ` with `hk : i < k ∧ k < j`,
this lemma **can only be applied when there exists some k between i and j**.
For adjacent pairs `j = i+1`, **no such k exists**, and `source_exists` cannot
be applied at all.

```lean4
-- This works (j-i ≥ 2, k exists):
have hex_k : ∃ k, i < k ∧ k < j := by
  have : i + 1 < j := by omega
  exact ⟨i+1, by omega, by omega⟩
rcases hex_k with ⟨k, hk, hk'⟩
have h_src := source_exists D i j u v … k hk hk' h2ne

-- This is IMPOSSIBLE (j = i+1, no k exists):
-- No k with i < k < i+1
```

### The adjacent-pair gap and why IH doesn't rescue it

When `j = i+1` (adjacent), you might try to decompose a **larger pair** `(i,u)`
where `u > i+1`:

```lean4
-- The identity 2D(E_{i,u}) = [D(E_{i,i+1}), E_{i+1,u}] + [E_{i,i+1}, D(E_{i+1,u})]
-- gives: (2:F)*D.f i u (i+1) v = -D.f i (i+1) u v
```

This involves `D.f i u (i+1) v` (coefficient in D(E_{i,u})). Target length
`v-(i+1)` vs original `d = v-u`:

```
v-(i+1) - (v-u) = u-(i+1) > 0    →    v-(i+1) > d
```

The target length is **LONGER**, so the IH doesn't apply.

### Phase VA pattern: resolving coupled gap chains with descending/ascending induction

When a proof has a gap at a specific case (e.g., adjacent sources `j=i+1`) that creates
a **chain of dependent gaps** (each gap depends on its neighbor), the resolution forms
its own proof phase with a specific structure.

#### Diagnosing chained vs. isolated gaps

Before writing any code, determine if the gap is **isolated** or **systemic**:

| Question | Isolated gap | Chained gap |
|----------|-------------|-------------|
| Cross-terms depend on same category? | No — cross-term is fully known | Yes — cross-term is in the SAME gap category |
| Proof by single equation? | Yes, one cond_zero or one bracket | No, requires induction across neighbors |
| Boundary cases separable? | N/A | Yes — two boundaries, interior handled by chain |

**Diagnostic test:** Apply the bracket/equation that SHOULD constrain the gap and
expand BOTH sides. Check what the cross-terms are:

- If cross-terms involve **fully known** quantities (non-adjacent sources, `j-i ≥ 2`),
  the gap is **isolated** — one equation suffices.
- If cross-terms involve the **same type** of unknown (adjacent sources at `i-1` or `i+1`),
  the gap is **chained** — needs induction.
  
**Example:** Right bracket `[E_{i,i+1}, E_{i+1,i+2}] = E_{i,i+2}` for eliminating
Type A coefficients `D.f i(i+1) u(i+1)`:

```
2·D(E_{i,i+2}) = [D(E_{i,i+1}), E_{i+1,i+2}] + [E_{i,i+1}, D(E_{i+1,i+2})]
     ↓ known                     ↓ gives Type_A(i)           ↓ T4 = Type_A(i+1)
```

T4 is Type_A(i+1) — same class of unknown, at neighbor `i+1`. **Chained gap.**

#### Structure for chained gaps: "Phase VA" pattern

A chained gap resolution module has seven parts:

```
1. Theorem statement (what the phase takes as input, what it outputs)
2. Boundary case analysis (smallest/largest index, degenerate situations)
3. Type-level decomposition (define Type_A, Type_B, Type_C as explicit defs)
4. Interior elimination lemmas (one per chain direction)
5. Boundary elimination lemmas (separate arguments for each boundary)
6. Combination lemma (all Type_A = 0, all Type_B = 0)
7. Final adjacent classification theorem
```

**Order of proof matters — find the induction direction that works:**

| Bracket | Eliminates | Cross-term | Induction direction |
|---------|-----------|------------|-------------------|
| Right: `[E_{i,i+1}, E_{i+1,i+2}]` | Type_A(i) | Type_A(i+1) | **Descending** (n-2 → 1) |
| Left: `[E_{i-1,i}, E_{i,i+1}]` | Type_B(i) | Type_A(i-1) | **Ascending** (2 → n-1) |

**Why descending works for right bracket:** When proving Type_A(i) = 0, the T4
cross-term is Type_A(i+1). If we prove Type_A(n-2) FIRST (the last i for which
the right bracket exists), Type_A(n-2) has no T4 issue (Type_A(n-1) is a
boundary, handled separately). Then Type_A(n-3) uses Type_A(n-2) which we
already know. Continue down to 1.

```lean4
-- Descending induction pattern (CORRECT Nat.decreasingInduction usage):
lemma right_bracket_typeA (D : NCoeff F n) (c : F)
    (h_nonadj : ∀ i j u v, j-i ≥ 2 → … → D.f i j u v = 0) (h_diag : … → D.f i j i j = c)
    (i : ℕ) (hi : 1 ≤ i) (hin : i+2 ≤ n) : ∀ u, 1 ≤ u → u < i → typeA D i u = 0 := by
  -- Define the property to prove by descending induction
  let motive (m : ℕ) (hm : m ≤ n-2) : Prop :=
    1 ≤ m → m+2 ≤ n → ∀ u, 1 ≤ u → u < m → typeA D m u = 0
  have h_base : motive (n-2) (by omega) := by
    intro hn2_pos hn2_bound u hu hun2
    -- Handle Type_A(n-2) — needs Type_A(n-1) which is a boundary case
    -- Boundary requires separate argument (cond_zero or alternative bracket)
    …
  have h_step : ∀ k, k < n-2 → motive (k+1) (by omega) → motive k (by omega) := by
    intro k hk h_next
    intro hk_pos hk_bound u hu huk
    unfold motive at h_next
    -- h_next : 1 ≤ k+1 → (k+1)+2 ≤ n → ∀ u, 1 ≤ u → u < k+1 → typeA D (k+1) u = 0
    -- Use D.cond with source (k, k+2), target (u, k+2), split at k+1:
    -- T1 = typeA D k u (our target)
    -- T4 = typeA D (k+1) k (zero by h_next, giving us the induction step)
    …
  have hi_bound : i ≤ n-2 := by omega
  have h_result := Nat.decreasingInduction h_step h_base hi_bound
  exact h_result hi hin
```

**⚠️ What NOT to do** — this `revert i / refine Nat.decreasingInduction ?_ (n-2)` syntax is
**wrong** and causes "failed to elaborate eliminator" errors:
```lean4
  -- ❌ WRONG: Nat.decreasingInduction ?_ (n-2) treats (n-2) as 'self' arg,
  --    but 'self' expects type 'motive n …', not ℕ
  revert i
  refine Nat.decreasingInduction ?_ (n-2)
```

```lean4
-- Ascending induction pattern (symmetrically):
lemma left_bracket_typeB (D : NCoeff F n) (c : F)
    (h_nonadj : …) (h_diag : …)
    (i : ℕ) (hi : 2 ≤ i) (hin : i+1 ≤ n) : ∀ v, i+1 < v → v ≤ n → typeB D i v = 0 := by
  -- Use regular strong induction (Nat.strong_induction_on) from 2 upward
  induction' i using Nat.strong_induction_on with i ih
  intro v hv hvn
  …
```

#### Boundary handling pattern

Boundaries (Type_A(n-1) and Type_B(1)) are NOT covered by the chain and require
separate arguments. The most reliable approach: use `cond_zero` with carefully
chosen `(c,d)` pairs such that:

1. `[E_{i,i+1}, E_{c,d}] = 0` (the bracket must be zero for cond_zero to apply)
2. One of the cond_zero four terms isolates the target coefficient
3. All other cond_zero terms are zero (known from nonadjacent classification)

**Finding the right (c,d) pair:** You want T1 or T3 or T2 or T4 in the cond_zero
equation to equal the coefficient you're trying to eliminate. For example:

To eliminate Type_A(n-1) = D.f (n-1)n u n:
- Try cond_zero(n-2,n, n-1,n, u,n): T1 = D.f (n-2)n u(n-1) (known 0 from nonadjacent),
  T3 = D.f (n-1)n (n-2)n (this IS Type_A(n-1) for u = n-2, when u < n-2 the term
  in T4 is D.f (n-1)n u (n-2), which is a different coefficient).
  
**The key condition:** `j ≠ c` and `d ≠ i` in cond_zero. Without these, cond_zero
doesn't apply.

**⚠️ `cond_zero` argument ordering:** The 9th and 10th arguments of `cond_zero`
expect `j ≠ c` and `d ≠ i` respectively (in that exact order). When you have
`h_i_ne_n : i ≠ n` and need to pass it as the `d ≠ i` argument (where `d = n`),
the expected type is `n ≠ i`, not `i ≠ n`. Use `Ne.symm h_i_ne_n`.
Passing `h_i_ne_n` directly gives: `Application type mismatch: i ≠ n vs expected n ≠ i`.

#### Type decomposition pattern

Define explicit `typeA`, `typeB`, `typeC` as `def`s to keep the proof organized:

```lean4
def typeA (D : NCoeff F n) (i u : ℕ) : F := D.f i (i+1) u (i+1)   -- u < i
def typeB (D : NCoeff F n) (i v : ℕ) : F := D.f i (i+1) i v       -- v > i+1
def typeC (D : NCoeff F n) (i u v : ℕ) : F := D.f i (i+1) u v    -- u < i, v > i+1
```

These defs make proof statements self-documenting and let you rewrite
`typeA D i u` back to `D.f i (i+1) u (i+1)` for the actual computation.

#### Cross-phase communication

The Phase VA module's output theorem signature should be:

```lean4
lemma adjacent_classification (c : F) (h_nonadj_full : ∀ i j, 1 ≤ i → i < j → j ≤ n →
    j - i ≥ 2 → ∀ u v, 1 ≤ u → u < v → v ≤ n → D.f i j u v = (if u = i ∧ v = j then c else 0)) :
    ∀ i, 1 ≤ i → i < n → ∀ u v, 1 ≤ u → u < v → v ≤ n → 
    D.f i (i+1) u v = (if u = i ∧ v = i+1 then c else 0) := ...
```

The input `h_nonadj_full` encapsulates both the nonadjacent diagonalization
(off-diagonal = 0) AND the classical classification (diagonal = c). This
single assumption is enough for both Type_C, Type_A, and Type_B elimination.

#### What to avoid

- **Don't try to prove each `Gap(i)` separately.** The chain structure means
  `Gap(i).proof` needs `Gap(i-1).proof` and `Gap(i+1).proof`, creating endless
  mutual recursion.
- **Don't use `cond_zero` with adjacent sources `(i,i+1)` as the first source**
  unless `(c,d)` is carefully chosen so one of the 4 if-conditions fires.
  `cond_zero(i,i+1,u,v,u,v)` gives `0=0` (all 4 conditions: `u < u`, `v < v`,
  `u = i`, `v = i+1` are ALL false for `u < i < i+1 < v`). **Type C cannot be
  proved via `cond_zero`** in this form — use nonadjacent-source `cond` embedding
  instead.
- **Don't extract `nonadjacent` as a separate lemma if it needs `h_ge2 : j-i ≥ 2`**
  in its parameters — it won't handle adjacent-source cross-terms from `source_exists`.
  Use the inner-`have` pattern instead (documented above).
- **Don't use `subst`** when the equality is between two binder variables —
  it can remove binders. Use `rw` instead.

The cleanest way to extract a reusable nonadjacent lemma inside a proof by strong
induction on target length: write it as a `have` block with its OWN strong
induction, BEFORE the main proof body.

```lean4
theorem diagonalization (D : NCoeff F n) (h2ne : (2 : F) ≠ 0) : … := by
  -- ✅ Inner lemma with its own strong induction, independent of the outer proof
  have h_nonadjacent : ∀ i j u v, 1 ≤ i → i < j → j ≤ n → j - i ≥ 2 →
      1 ≤ u → u < v → v ≤ n → (u ≠ i ∨ v ≠ j) → D.f i j u v = 0 := by
    intro i j u v hi hij hjn h_ge2 hu huv hvn h_off
    induction' hlen : v - u using Nat.strong_induction_on with d ih generalizing i j u v
    by_contra! h_nonzero
    -- … source_exists + 4 cases, each calling ih (which does NOT need h_ge2) …

  -- Main proof body
  intro i j u v hi hij hjn hu huv hvn h_off
  by_cases h_ge2 : j - i ≥ 2
  · exact h_nonadjacent i j u v hi hij hjn h_ge2 hu huv hvn h_off
  · -- adjacent case: use h_nonadjacent for cross-terms in cond_zero
    …
```

**Why this works:** The inner `induction'` captures `hlen : v-u = d` as the
induction hypothesis's measure. The IH `ih` covers ALL pairs with shorter
target length — it does NOT require `j-i ≥ 2`. This is crucial because
`source_exists` calls `ih` for pairs like `(i,k)` with `k = i+1` (adjacent,
`j-i = 1`), which would fail if the IH required `h_ge2`.

**Without this pattern**, extracting `nonadjacent` as a top-level lemma with
`h_ge2` in its signature causes IH failures: the IH for `(i,k)` requires
`k-i ≥ 2`, but `k = i+1` (the only choice when `j = i+2`) gives `k-i = 1`.

### Known workarounds for the adjacent-pair gap

1. **Backward induction on source index** (most promising):
   Prove D(E_{n-1,n}) first, then D(E_{n-2,n-1}), ..., D(E_{1,2}).
   At step `k`, use nonadjacent pairs `(k-1,k+1)` to constrain `(k,k+1)`:

   ```
   (k-1, k+1) decomposition with k = k:
   2D(E_{k-1,k+1}) = [D(E_{k-1,k}), E_{k,k+1}] + [E_{k-1,k}, D(E_{k,k+1})]
   D(E_{k-1,k+1}) is diagonal (nonadjacent, length 2).
   Coefficient of E_{k-1,v}: D.f k (k+1) (k-1) v appears in the RHS.
   ```

   This requires D(E_{k-1,k}) to be diagonal (which is the PREVIOUS step).

2. **Multi-step bracket chain**:
   Use a combination of nonadjacent pairs to propagate constraints.
   This is mathematically sound but complex to formalize.

3. **Paper-proof gap note**:
   Many published proofs of this form implicitly assume `j-i ≥ 2` without
   stating it. Check whether the paper actually handles adjacent pairs or
   if the main theorem only needs nonadjacent ones (e.g., the RecursiveSystem
   for n ≥ 5 works with C_{i,i+1} as free parameters that collapse via
   periodicty, bypassing the need for D(E_{i,i+1}) diagonal).

4. **Zero-bracket condition (`cond_zero`)** — BEST APPROACH:\n   Add a separate field to your structure that encodes the condition when\n   `[E_{ab}, E_{cd}] = 0`. The 1/2-derivation identity gives:\n   ```\n   0 = [D(E_{ab}), E_{cd}] + [E_{ab}, D(E_{cd})]\n   ```\n   Expanded in the basis {E_{uv}}, this becomes:\n   ```\n   δ_{v,d}·f a b u c (if u < c) - δ_{u,c}·f a b d v (if d < v)\n   + δ_{u,a}·f c d b v (if b < v) - δ_{v,b}·f c d u a (if u < a)\n   = 0\n   ```\n   \n   The `cond_zero` structure field encoding this:\n   ```lean4\n   structure NCoeff where\n     f : ℕ → ℕ → ℕ → ℕ → F\n     cond : ∀ i j u v, … → ∀ k, i < k → k < j → …\n     cond_zero : ∀ i j c d, 1 ≤ i → i < j → j ≤ n → 1 ≤ c → c < d → d ≤ n →\n       j ≠ c → d ≠ i → ∀ u v, 1 ≤ u → u < v → v ≤ n →\n       (if u < c ∧ v = d then f i j u c else 0)\n       - (if u = c ∧ d < v then f i j d v else 0)\n       + (if u = i ∧ j < v then f c d j v else 0)\n       - (if u < i ∧ v = j then f c d u i else 0) = 0\n   ```\n   \n   **Key technique for choosing (c,d) to isolate a coefficient:**\n   To isolate `D.f i(i+1) u v` for Type C (u < i < i+1 < v), choose\n   `(c,d) = (v, v+1)` with target `(u, v+1)`. Only Term 1 survives:\n   ```\n   (if u < v ∧ (v+1) = (v+1) then D.f i(i+1) u v else 0) = D.f i(i+1) u v\n   ```\n   All other terms vanish. Requires `v < n` (interior).

#### Enlarged-pair pattern for Blind A/B coefficients (target disjoint from source)

When the target `(u,v)` is **completely before** the source `(i,i+1)` (i.e., `v < i`),
use `cond_zero` with sources `(i,i+1)` and `(v,n)` and target `(u,n)`. Term 1
isolates the target coefficient D.f i(i+1) u v. Term 4 gives a residual term
only when `i = n-1` (killed by h_nonadj). All other terms vanish.

When the target is **completely after** the source (i.e., `u > i+1`), use
`cond_zero(i,i+1,i-1,u,i-1,v)`. Term 2 gives -(target coefficient). All other
terms vanish. Requires `i ≥ 2`. For `i = 1`, use `cond_zero(1,2,1,u,1,v)` + h_nonadj.

#### Degree of freedom vs. proof gap — the bracket image diagnostic

When you're stuck on a `sorry` and suspect the coefficient might be a genuine
degree of freedom (not just a missing proof), use this diagnostic:

**Bracket image test:** For the target unit E(u,v), test it against ALL commuting
M in N_n (i.e., M such that [E(source), M] = 0). Compute [E(u,v), M] for each M.
If **every** non-zero bracket produces the SAME basis element (e.g., always
±E(1,n)), then the coefficient is a **structural corner exception** — no second
independent relation can be found via bracket identities.

If different M produce different basis elements, a missing proof is likely.

**Verification with known examples:** Once you identify a candidate structural
exception, verify by checking whether a known explicit example (like `centralPert`)
has this coefficient non-zero. If the known example has it zero, check whether
that's because the example lives in a special subspace — or because the coefficient
is forced to zero by a relation you haven't found yet.

**Resolution strategy for structural exceptions:**
1. Do NOT try to prove the coefficient is zero — it isn't.
2. Add an explicit hypothesis to the theorem excluding the exception pair (e.g., `h_not_coupled : ¬(i = n-1 ∧ u = 2 ∧ v = n) ∨ ¬(i = 1 ∧ u = 1 ∧ v = n-1)`).
3. In the proof dispatch, use `h_not_coupled` to derive a contradiction when the exception case arises.
4. In the classification theorem, handle the exception via its known relation (e.g., `corner_relation : a = -b`).

This is preferable to a false vanishing claim. The theorem stays honest, the
exception becomes part of the classification structure (e.g., Hom(L/[L,L], Z(L))).

**Example walkthrough — the coupled corner pair for N_n:**

```
Coefficients: a = D.f (n-1)n 2 n, b = D.f 1 2 1 (n-1)
Known:       corner_relation gives a = -b
Bracket test: For any M with [E(n-1,n), M] = 0:
  [E(2,n), M] ≠ 0 only when M = E(1,2), and [E(2,n), E(1,2)] = -E(1,n)
  [E(1,n-1), M] ≠ 0 only when M = E(n-1,n), and [E(1,n-1), E(n-1,n)] = -E(1,n)
Both coefficients' non-zero brackets always land on E(1,n) — the center.
→ No second independent relation exists → structural exception.
```

**When to use:** When a `sorry` survives multiple attempts and all cond_zero
instances give 0 = 0. Before labeling it a structural exception, verify by
the bracket image test AND check against at least one known example (like
centralPert).

#### `h_not_coupled` pattern for honest classification theorem statements

When a structural exception is confirmed, the theorem must be modified to
ACKNOWLEDGE it, not suppress it. The pattern:

1. Add a hypothesis to the theorem that excludes the exception pair:
   ```lean4
   theorem adjacent_reduce_to_corner (D : NCoeff F n) (h2ne : ...) ...
       (h_not_corner : u ≠ 1 ∨ v ≠ n)
       (h_not_coupled : ¬(i = n-1 ∧ u = 2 ∧ v = n) ∨ ¬(i = 1 ∧ u = 1 ∧ v = n-1)) :
       D.f i (i+1) u v = 0 := by
   ```
   The `h_not_coupled` hypothesis says: this call is NOT asking about the
   coupled corner pair. If it IS asking about that pair, the hypothesis becomes
   `False ∨ True` = `False`, making the case unreachable.

2. In the proof dispatch, when the exception case arises, derive a contradiction
   from `h_not_coupled`:
   ```lean4
   · -- u = 2: coupled corner pair (i=n-1, u=2, v=n)
     have hv_n : v = n := rfl  -- after subst, v=n
     rcases h_not_coupled with (h_excl | _)
     · exfalso; exact h_excl ⟨rfl, hu2, hv_n⟩
     · exfalso; have hi_ne_1 : i ≠ 1 := by omega; exact hi_ne_1 rfl
   ```

3. In the classification theorem, handle the exception via its known relation:
   ```lean4
   -- The coupled pair satisfies: D.f (n-1)n 2 n = -D.f 1 2 1 (n-1)
   -- (corner_relation). This is part of the Hom(L/[L,L], Z(L)) structure,
   -- not an error in the vanishing proof.
   ```

**Key rule:** Never claim `D.f (...) = 0` when you know a counterexample exists
(e.g., `centralPert` has a non-zero value). The `h_not_coupled` pattern keeps
the theorem honest while still allowing callers who fall outside the exception
case to use the theorem. The classification layer then handles the exception
via its structural relation.

**⚠️ `∨` vs `∧` in the `h_not_coupled` encoding — a correctness trap.**

When the exception involves TWO distinct corner cases (e.g., `(i=n-1, u=2, v=n)`
and `(i=1, u=1, v=n-1)`), the TYPE of `h_not_coupled` MUST be `¬(A ∨ B)` (one `¬`
over the `∨`), NOT `¬A ∨ ¬B` (two `¬`s with `∨`). Here's why:

```lean4
-- ❌ WRONG: ¬A ∨ ¬B doesn't exclude A when B is trivially true
h_not_coupled : ¬(i = n-1 ∧ u = 2 ∧ v = n) ∨ ¬(i = 1 ∧ u = 1 ∧ v = n-1)

-- When i=n-1 (so A = True ∧ u=2 ∧ True = u=2):
--   ¬A = ¬(u=2),  ¬B = ¬(n-1=1 ∧ u=1 ∧ n=n-1) = True (for n≥4)
--   h_not_coupled = ¬(u=2) ∨ True = True  ← does NOT exclude u=2!
--   Caller with u=2 falls through the ∨ to the ¬B branch, which is trivially true
--   → cannot derive contradiction → STUCK

-- ✅ CORRECT: ¬(A ∨ B) excludes both simultaneously
h_not_coupled : ¬((i = n-1 ∧ u = 2 ∧ v = n) ∨ (i = 1 ∧ u = 1 ∧ v = n-1))
```

**When `¬(A) ∨ ¬(B)` breaks:** The `∨` creates a reachable case where the
second disjunct is trivially true (e.g., `¬(n-1=1 ∧ …)` for `n≥4`) even though
the first is false (`u=2` contradicts `¬(u=2)`). The caller intended to EXCLUDE
the coupled pair, but the `¬(A) ∨ ¬(B)` form lets them satisfy the hypothesis
via the trivial `¬B` even when `A` is true. The `rcases h_not_coupled with (h | h)`
then enters the `¬B` branch, which carries `True` — no contradiction derivable.

**Fix:** Change to `¬(A ∨ B)` and use direct `apply` or `simp`:

```lean4
h_not_coupled : ¬((i = n-1 ∧ u = 2 ∧ v = n) ∨ (i = 1 ∧ u = 1 ∧ v = n-1))

-- Inside proof, when i=n-1, v=n, u=2:
have h_no_u2 : u ≠ 2 := by
  intro h_u2
  apply h_not_coupled
  left; exact ⟨rfl, h_u2, rfl⟩  -- i=n-1, u=2, v=n — excluded!
```

**Boundary case — `h_not_coupled` after `subst`:** When `subst` has replaced
`i` and `u` with concrete values, `h_not_coupled` simplifies. For `i = n-1, u = 2,
v = n`:
- First disjunct: `¬(n-1 = n-1 ∧ 2 = 2 ∧ n = n)` = `¬True` = `False`
- Second disjunct: `¬(n-1 = 1 ∧ 2 = 1 ∧ n = n-1)` = `¬False` = `True`
So `h_not_coupled` = `False ∨ True` = `True`. `rcases (...) with (h_excl | _)`
gives `h_excl : False`, which closes the goal by `exfalso; exact h_excl`.

For `i = 1, u = 1, v = n-1`:
- First disjunct: `¬(1 = n-1 ∧ 1 = 2 ∧ n-1 = n)` = `¬False` = `True`
- Second disjunct: `¬(1 = 1 ∧ 1 = 1 ∧ n-1 = n-1)` = `¬True` = `False`
`rcases (...) with (_ | h_excl)` gives `h_excl : False`, same resolution.

### Key algebra: simplifying D.cond terms for adjacent-specific pairs

When you DO have `j-i ≥ 2` and need to manually compute which `D.cond` terms
contribute:

```lean4
have h_id := D.cond i j u v hi hij hjn hu huv hvn k hk hk'
-- Goal: compute each of the 4 if-then-else terms for YOUR specific indices.

-- Pattern: use simpa with explicit simp lemmas:
have h_eq : (2 : F) * D.f i u (i+1) v = -(D.f i (i+1) u v) := by
  have h_simp1 : (if v = u ∧ (i+1 : ℕ) < (i+1 : ℕ) then … else 0) = 0 := by simp
  have h_simp2 : (if (i+1 : ℕ) = (i+1 : ℕ) ∧ u < v then D.f i (i+1) u v else 0) = D.f i (i+1) u v := by
    simp [huv]
  have h_simp3 : (if v = (i+1 : ℕ) ∧ (i+1 : ℕ) < i then … else 0) = 0 := by simp
  have h_simp4 : (if (i+1 : ℕ) = i ∧ (i+1 : ℕ) < v then … else 0) = 0 := by simp
  simpa [h_simp1, h_simp2, h_simp3, h_simp4] using h_id
```

### Common `simpa` result patterns for D.cond

Given `(i, j, u, v, k)`:

**Pattern A — IB term gives -target:** When `u = k ∧ j < v` is the only
contributing term:
```
(2:F)*D.f i j u v = -D.f i k j v     (IB gives f i k j v, with negative sign)
```

**Pattern B — IIB term gives +target:** When `u = i ∧ k < v` is the only
contributing term:
```
(2:F)*D.f i j u v = +D.f k j k v     (IIB gives f k j k v, with positive sign)
```

**Pattern C — All zero:** When the target indices don't match any source
decomposition pattern (happens for certain DIFFERENT pair combinations):
```
(2:F)*D.f i j u v = 0                → D.f i j u v = 0 (since 2≠0)
```

#### `split_ifs` naming conflict with `h`

**Problem:** `split_ifs` with `h` as the binder name can create a naming conflict
when `h` is already used as a hypothesis. In the `False` branch of `split_ifs`,
`h` becomes a function `¬(condition)` rather than a destructurable proposition.
Accessing `h.1` then fails because `.1` doesn't apply to function types:

```lean4
by_cases hn_eq_ip1 : n = i+1
· have h_T4 : (if u < i ∧ n = (i+1 : ℕ) then D.f v n u i else 0) = D.f v n u i := by
    have h_ui : u < i := lt_trans huv hvi
    split_ifs with h
    · rfl
    · exfalso; exact h.1 h_ui   -- ❌ error: h has function type, can't project .1
```

**Fix:** Use `by_cases` instead of `split_ifs`:

```lean4
    by_cases h_cond : u < i ∧ n = (i+1 : ℕ)
    · simp [h_cond]    -- h_cond is And, accessible via h_cond.1, h_cond.2
    · simp [h_cond]    -- h_cond is ¬(…), usable as h_cond : ¬(condition)
```

Or, if you KNOW the condition is true in context, construct the condition directly:

```lean4
    have h_cond : u < i ∧ n = (i+1 : ℕ) := ⟨lt_trans huv hvi, hn_eq_ip1⟩
    simp [h_cond]
```

See `references/split-ifs-ring-pattern.md` for the `split_ifs` + `ring` technique for proving linear conditions.

**Problem:** `subst h` replaces a variable everywhere, including in hypotheses
and the binder list. If the equality is between two already-existing binders
(e.g., `h : u = i`), `subst` can make the outer binder `i` disappear from
the context, causing "Unknown identifier `i`" errors.

**Fix:** Use `rw [h]` (at the top of the block) instead of `subst h`. This
rewrites `u` to `i` in the goal and all hypotheses WITHOUT removing the
binder `u` from the context:

```lean4
-- ❌ subst removes binder, causes 'unknown identifier' on later i references
subst hu_eq_i

-- ✅ rw rewrites without removing the binder
rw [hu_eq_i]
```

This is more reliable when you need to keep referencing `i` later in the
proof block (e.g., in `calc` chains or `omega` calls where `i` is mentioned).

#### `subst` does NOT rewrite hypothesis types (only the goal)

**Critical Lean 4 behavior difference from Lean 3:** `subst h` (where `h : a = b`) rewrites `a` to `b` **in the goal only**. Unlike Lean 3's `subst`, Lean 4's `subst` does **not** rewrite the types of hypotheses in the context. This is a documented design decision — `subst` uses `hcase` internally, which only affects the target.

**Symptom:** After `subst hvi_eq` (where `hvi_eq : v = i`), the hypothesis `huv : u < v` still has type `u < v` (not `u < i`). Attempting `have h_u_lt_i : u < i := huv` gives a type mismatch: `huv` has `u < v` but `u < i` was expected.

```lean4
-- ❌ fails: huv still has type u < v, not u < i
have hvi_eq : v = i := by omega
subst hvi_eq
have h_u_lt_i : u < i := huv  -- Type mismatch: huv : u < v
```

**Fix 1 — prove BEFORE `subst`:** Establish all dependent facts before the substitution:

```lean4
-- ✅ BEFORE subst: prove using both hypotheses
have hvi_eq : v = i := by omega
have h_u_lt_i : u < i := calc
  u < v := huv
  _ = i := hvi_eq     -- uses hvi_eq to connect
subst hvi_eq           -- now v is gone, but h_u_lt_i is already established
```

**Fix 2 — `rw` instead of `subst`:** Use `rw [hvi_eq]` which rewrites `v` to `i` in the goal WITHOUT removing the `v` binder:

```lean4
rw [hvi_eq]  
-- v is still in the binder list, but the goal now uses `i` instead of `v`
-- huv also remains as u < v (unrewritten), but you can still use huv directly
```

**Fix 3 — `rw` with `at` and `⊢`:** Rewrite in specific hypotheses and the goal simultaneously:

```lean4
rw [hvi_eq] at huv ⊢
-- huv : u < i  (rewritten)
-- goal: D.f i (i+1) u i = 0  (rewritten, v = i in goal)
```

**Why `subst + open Struct` makes it worse:** When `open MatIdx` is active (exposing `.i : MatIdx → ℕ`), `subst h : u = i` can make the binder `i` resolve to the **projection function** `MatIdx.i` in hypotheses. Since `subst` only rewrites the goal but the projection ambiguity affects how `omega` resolves `i` in hypothesis types, even `omega` calls after `subst` may fail with "don't know how to synthesize implicit argument".

**Rule of thumb:** If you need a hypothesis to carry the substitution forward, use `rw` or `calc` BEFORE `subst`. If you only need the goal rewritten, `rw` alone is safer than `subst`.

#### `subst` with `open Struct` projection collision (batch rewrite)

When `open SomeStruct` is active (e.g., `open MatIdx` exposes `.i : MatIdx → ℕ`),
`subst h` where `h : u = i` can make `i` resolve to the **projection function**
rather than the binder `i : ℕ`. The symptom: after `subst`, `i` is treated as
`MatIdx → ℕ` and the goal `i+1 < v` fails with "don't know how to synthesize
implicit argument `α`" (since `i` is now a function, not a `ℕ`).

**Fix 1 (preferred):** Instead of `subst h_u_eq_i`, use `subst u` (substituting the
OTHER variable by name):

```lean4
-- ❌ subst h_u_eq_i makes i resolve to MatIdx.i projection
subst h_u_eq_i

-- ✅ subst u avoids the projection collision — u is a plain ℕ, not a struct field
subst u
```

**Fix 2:** Simplify `h_u_eq_i` by rewriting `u` to `i` in all hypotheses and the goal,
without substituting the binder `i` away:

```lean4
rw [h_u_eq_i]
-- rewrites u to i everywhere; u remains in the binder list
```

**Fix 3:** Use `rw [h_u_eq_i]` at specific hypotheses instead of globally.
If you only need `i` and `u` to be equal in one place, rewrite there.

**Also affects `subst` with `v = n`** — when `hvn_eq_n : v = n` and `n` is a binder,
`subst hvn_eq_n` may replace `n` with `v` (or vice versa) in a way that makes `n`
unavailable by name. Use `rw [hvn_eq_n]` instead to rewrite `v` to `n` in specific
hypotheses or the goal, without removing the `n` binder.

**⚠️ Nested `subst` can remove outer binder variables silently.** When you have
`h_n_eq_ip1 : n = i+1` inside a proof block, `subst h_n_eq_ip1` replaces `n`
with `i+1` EVERYWHERE — including in hypotheses and subsequent goals. If the
code after `subst` references `n` (e.g., `have hn_ge_4 : 4 ≤ n := ...`), Lean
reports `Unknown identifier 'n'`. **Fix:** use `rw [h_n_eq_ip1]` instead, or
delete the `subst` line if the block is a frozen `sorry` (boundary case). When
`sorry` blocks sit inside `subst`-modified contexts, removing the `subst` is
the minimal change that preserves the rest of the proof.

#### `structure where` block: field `f` is NOT unfoldable via `unfold f` or `simp [f]`

Inside a `structure MyStruct where f := ...` block, the field `f` is defined as part of the
structure instance being constructed — it is **not** a local definition that `unfold` or
`simp [f]` can access. The error message is `Unknown identifier f`.

```lean4
noncomputable def toNCoeff (D : ...) : NCoeff F n where
  f := λ i j u v => D.coeff ... ...
  cond := by
    intro ...
    -- ❌ unfold f  → error "Unknown identifier f"
    -- ❌ simp [f]  → error "Unknown identifier f"
    -- The field `f` is NOT in scope as a local binder
    
    -- ✅ Use the definition directly:
    -- The goal already has `f i j u v` but `unfold`/`simp` won't expand it.
    -- Workaround 1: Use `change` to manually rewrite the goal
    change (2 : F) * D.coeff ⟨i,j,hi,hij,hjn⟩ ⟨u,v,hu,huv,hvn⟩ = ...
    -- Workaround 2: Leave `f` as-is and prove the goal using a lemma that connects f to coeff.
    -- Workaround 3: Accept the gap and leave `sorry` with a note
```

**Root cause:** In `structure where` syntax, the `f := ...` clause defines a projection of the
resulting structure. The `cond` and `cond_zero` clauses are proofs ABOUT this projection,
but the projection name `f` is not available for `unfold`/`simp` during the construction.
The field IS available as a `let`-like binder in `calc` and `have` via `this.f` in some
versions of Lean, but this is unreliable across versions.

**Best practice:** If you need to expand `f` inside a `where` block, write the proof as
a separate lemma that takes `f` as an explicit hypothesis, then call it from the `where`
block:

```lean4
lemma cond_proof (f : ℕ → ℕ → ℕ → ℕ → F)
    (half_deriv_cond : ∀ (x y : MatIdx n) (k : ℕ), ... → (2 : F) * f ... = ... ) : ... := by
  -- f is accessible here as a binder — unfold, simp, rfl all work
  ...

noncomputable def toNCoeff (D : ...) : NCoeff F n where
  f := ...
  cond := cond_proof (toNCoeff D).f D.half_deriv_cond ...
```

This separates the concern: the external lemma has full access to `f`, and the
`where` block just calls it.

Cleaner alternative: `coeffOf` + external lemma pattern. Define the function
as a standalone `def` BEFORE the `structure where`, prove lemmas about it, then
assemble in the `where` block:

```lean4
noncomputable def coeffOf (D : HalfDerivation F n) (i j u v : ℕ) : F :=
  if hi : 1 ≤ i then ... else 0

lemma coeffOf_f (D : ...) (i j u v : ℕ) ... : coeffOf D i j u v = D.coeff ... := by
  unfold coeffOf; simp [...]

lemma coeffOf_cond (D : ...) (i j u v : ℕ) ... (k : ℕ) ... :
    (2 : F) * coeffOf D i j u v = ... := by
  -- `coeffOf` is a plain def — `unfold`, `simp` all work
  ...

noncomputable def toNCoeff (D : HalfDerivation F n) : NCoeff F n where
  f := coeffOf D
  cond := coeffOf_cond D
  cond_zero := coeffOf_cond_zero D
```

**Same issue applies to anonymous constructor `{ field := ...; prop_field := ... }`**
**as well as named `structure where` blocks.** Inside `{ coeff := λ x y => ...;
half_deriv_cond := by ... }`, `dsimp [coeff]` and `simp [coeff]` BOTH fail with
"made no progress" because `coeff` is a field projection of the anonymous instance,
**Same issue applies to anonymous constructor `{ field := ...; prop_field := ... }`**  
**as well as named `structure where` blocks.** Inside `{ coeff := λ x y => ...; half_deriv_cond := by ... }`, `dsimp [coeff]` and `simp [coeff]` BOTH fail with "made no progress" because `coeff` is a field projection of the anonymous instance, not a standalone definition. The fix is identical: extract the field value into a standalone `def`/`let` before the constructor:

```lean4
-- ❌ dsimp/simp on coeff inside anonymous constructor fails
noncomputable def T (D : HalfDerivation F n) : HalfDerivation F n :=
  { coeff := λ x y => D.coeff x y - c * ((idHalfDeriv F n).coeff x y)
    half_deriv_cond := by
      intro x y k hk hk'
      dsimp [coeff]  -- ❌ "made no progress"
      ...
  }

-- ✅ Define coeff separately as a standalone def
noncomputable def T_coeff (D : HalfDerivation F n) : MatIdx n → MatIdx n → F :=
  λ x y => D.coeff x y - c * ((idHalfDeriv F n).coeff x y)

noncomputable def T (D : HalfDerivation F n) : HalfDerivation F n :=
  { coeff := T_coeff D
    half_deriv_cond := by
      intro x y k hk hk'
      dsimp [T_coeff]  -- ✅ T_coeff is a named def — dsimp works
      ...
  }

-- ✅ Even better: `let` + `have` pattern — avoids writing huge `forall` type
noncomputable def T (D : HalfDerivation F n) : HalfDerivation F n :=
  let Tcoeff : MatIdx n → MatIdx n → F := λ x y => D.coeff x y - c * ((idHalfDeriv F n).coeff x y)
  have h_half_deriv := by
    intro x y k hk hk'
    dsimp [Tcoeff]  -- `let` binder is dsimp-able
    ... proof using hD, hId, h_if_distrib, ring ...
  { coeff := Tcoeff
    half_deriv_cond := h_half_deriv
  }
```

The `let Tcoeff ... + have h_half_deriv := ...` pattern has two advantages:
1. `dsimp [Tcoeff]` works because `Tcoeff` is a `let` binder (local definition).
2. The `have` lemma's type is **inferred**, avoiding timeouts from huge `forall` types.

For HalfDerivation and similar structures with large field-condition types,
this prevents heartbeat limits during type-checking.

### half_deriv_cond linearity proof (calc + h_if_distrib + ring)

When defining `T = D - c*Id` (a linear combination), the condition field proof
follows a universal pattern since the condition is linear in `coeff`:

```lean4
let h_sc := c
have hD := D.half_deriv_cond x y k hk hk'
have hId := (idHalfDeriv F n).half_deriv_cond x y k hk hk'

-- Key lemma: each `if cnd then Tcoeff ... else 0` expands linearly
have h_if_distrib (cnd : Prop) [Decidable cnd] {a b : MatIdx n} :
    (if cnd then D.coeff a b - h_sc * ((idHalfDeriv F n).coeff a b) else 0)
    = (if cnd then D.coeff a b else 0)
    - h_sc * (if cnd then (idHalfDeriv F n).coeff a b else 0) := by
  split <;> simp

dsimp [Tcoeff]
-- Goal: 2*(D.coeff - h_sc*Id.coeff) = RHS

-- Step 1: ring on LHS
have h_expand : 2*(D.coeff x y - h_sc*Id.coeff x y) = (2*D.coeff x y) - h_sc*(2*Id.coeff x y) := by ring
rw [h_expand]
-- Step 2: replace 2*D.coeff and 2*Id.coeff with their RHS via hD, hId
rw [hD, hId]
-- Step 3: expand each Tcoeff term linearly
repeat (rw [h_if_distrib])
-- Step 4: ring to close
ring
```

The `h_if_distrib` lemma is the only structure-dependent part.

**⚠️ Pitfall: `rw` + `h_if_distrib` fails on `dite` terms with complex binders.**
When the RHS's `ite`/`dite` terms involve complex MatIdx constructor expressions
(e.g., `x.left k hk hk'` producing a `MatIdx n` with 3 embedded proofs), `rw`
often cannot match the implicit `a b : MatIdx n` arguments in `h_if_distrib`.
The `repeat (rw [h_if_distrib])` silently does nothing, and the subsequent
`ring` sees an unreduced goal. **Reliable alternative: `split_ifs` + `ring`:**

```lean4
have h_half_deriv := by
  intro x y k hk hk'
  dsimp [Tcoeff]
  have hD := D.half_deriv_cond x y k hk hk'
  have hId := (idHalfDeriv F n).half_deriv_cond x y k hk hk'
  have h_expand : (2 : F) * (D.coeff x y - c * Id.coeff x y)
      = ((2 : F) * D.coeff x y) - c * ((2 : F) * Id.coeff x y) := by ring
  rw [h_expand, hD, hId]
  split_ifs
  · ring
  · ring
  ... -- 16 cases total (4 independent dite conditions)
```

`split_ifs` resolves all `dite` terms into concrete values. Each of the 16
subgoals is a pure ring identity over the field F, which `ring` closes.
This is verbose (16 blocks) but completely reliable — no pattern-matching
issues with implicit MatIdx arguments. The 4 conditions come from the
standard half-derivation RHS: `y.j = x.j ∧ y.i < k`, `y.i = k ∧ x.j < y.j`,
`y.j = k ∧ y.i < x.i`, `y.i = x.i ∧ k < y.j`.
```

**Deep dive:** See `references/dite-to-ite-bridge.md` for a complete walkthrough of the `coeffOf` pattern with `dite`→`ite` conversion, including the 16-case `by_cases` pattern, MatIdx proof-irrelevance handling, and the `coeffOf_f` wrapper lemma.

**The 16-case `by_cases` pattern for dite/ite conversion.** When two expressions
differ only in `dite` (dependent if) vs `ite` (regular if), split on each condition
independently. Each branch `simp`s with the accumulated conditions, which resolves
both the `dite` and the `ite` to the same then/else value:

```lean4
-- Goal: (if h : v=j ∧ u<k then dite_term else 0) - ... = (if v=j ∧ u<k then ite_term else 0) - ...
by_cases h1 : v = j ∧ u < k
· rcases h1 with ⟨hvj, huk⟩; simp [hvj, huk, coeffOf, hi, hk, ...]
  by_cases h2 : u = k ∧ j < v
  · rcases h2 with ⟨huk2, hjv⟩; simp [huk2, hjv, coeffOf, hi, hk, ...]
    by_cases h3 : v = k ∧ u < i
    · rcases h3 with ⟨hvk, hui⟩; simp [hvk, hui, coeffOf, ...]
      by_cases h4 : u = i ∧ k < v
      · rcases h4 with ⟨hui2, hkv⟩; simp [hui2, hkv, coeffOf, ...]
        rfl  -- all 4 true: both sides simplify to same D.coeff
      · simp [h4, coeffOf, ...]
    · simp [h3, coeffOf, ...]
  · simp [h2, coeffOf, ...]
· simp [h1, coeffOf, ...]
```

This creates up to 16 subgoals (one per condition combination), but each is
handled uniformly: `simp` with the condition hypotheses resolves the `dite` to
a concrete value, and the `ite` + `coeffOf` to the same concrete value.

#### The `← neg_eq_zero` pattern for `x = 0 from -(x) = 0`

A concise way to prove `x = 0` from an equation giving `-(x) = 0`:

```lean4
-- Given: h_eq : (2:F) * D.f i u (i+1) v = -(D.f i (i+1) u v)
-- and: h_nad : D.f i u (i+1) v = 0
-- Want: D.f i (i+1) u v = 0

have h_zero : D.f i (i+1) u v = 0 := by
  rw [← neg_eq_zero, ← h_eq, h_nad, mul_zero]
  --        ^^^^^^^^^
  -- neg_eq_zero: x = 0 ↔ -x = 0
```

This avoids `ring`/`calc` chains for simple negated-zero proofs.
Same technique works for `-(a) = b` → `a = -b` via `neg_eq`:
`rw [← neg_eq_zero, ← h_eq, h_nad, mul_zero]`.

### Using `ite_eq_right_iff.mpr` when `simp` can't handle `¬P`

`simp` does **not** use `h : ¬P` to rewrite `if P then A else B = B`, even when `h` is in the `simp` set. This is a known limitation: `simp` only uses `¬h` for `¬(a = b)` to rewrite `a = b → False`, but this rule only fires when `¬P` appears syntactically as a `simp` lemma for the exact formula `P` inside the `ite`.

**Fix:** Use `ite_eq_right_iff.mpr` instead:

```lean4
-- ❌ simp doesn't use h_cond : ¬(u < i ∧ (v+1) = (i+1))
have h_term : (if u < i ∧ (v+1 : ℕ) = (i+1 : ℕ) then D.f v (v+1) u i else 0) = 0 := by
  simp [h_cond]   -- "h_cond is unused" warning

-- ✅ ite_eq_right_iff.mpr works
have h_term : (if u < i ∧ (v+1 : ℕ) = (i+1 : ℕ) then D.f v (v+1) u i else 0) = 0 := by
  apply ite_eq_right_iff.mpr
  intro h
  rcases h with ⟨h_ui, h_eq⟩
  have h_contra : v + 1 ≠ i + 1 := by omega
  exact absurd h_eq h_contra
```

The lemma `ite_eq_right_iff.mpr` has type `(P → a = b) → (ite P a b = b)`. Since `P` is false, the implication `P → a = b` is vacuously true, and the function body discharges it.

### The `Nat.ne_of_lt` ↔ `Ne.symm` direction gotcha

When using `simp` with `h : a ≠ b` to rewrite `a = b` to `False`, `simp` only matches the **exact syntactic form**:

```lean4
-- ✅ Works: h : x.i ≠ x.j
have h_ne : x.i ≠ x.j := Nat.ne_of_lt x.hlt
simp [h_ne]    -- rewrites x.i = x.j to False

-- ❌ Does NOT work: h_ne_symm : x.j ≠ x.i
have h_ne_symm : x.j ≠ x.i := Ne.symm (Nat.ne_of_lt x.hlt)
simp [h_ne_symm]   -- "h_ne_symm is unused" — can't rewrite x.i = x.j
```

`simp` matches the exact order of the equality in the `Ne` lemma. If the goal has `x.i = x.j`, use `Nat.ne_of_lt x.hlt` (which is `x.i ≠ x.j`), NOT `Ne.symm`. If the goal has `x.j = x.i`, use `Ne.symm (Nat.ne_of_lt x.hlt)`.

When in doubt, provide BOTH:
```lean4
simp [Nat.ne_of_lt x.hlt, Ne.symm (Nat.ne_of_lt x.hlt)]
```

## Error Priority

修复错误时**严格按此顺序**——高优先级错误会使低优先级错误的信息不可靠：

```
1. Syntax errors（语法错误）→
2. Type errors（类型错误）→
3. Unsolved goals / tactic failures（未解目标）→
4. Linter warnings（linter 警告）
```

**关键规则：**
- 「Unsolved goals」错误出现在 `by` 或 `=>` 行，**而不是你添加 tactic 的地方**
- 如果在 59 行有 "unsolved goals" 但在 65 行有 tactic 错误——**先修第 65 行**
- 遇到任何错误就停止写更多 tactic

## Hardest Case First

### 跨定理

直接攻克目标定理。**不要先填辅助引理中的 `sorry`**——Lean 把 `sorry` 当作公理，依赖它的定理仍能编译。

通过将 `sorry` 证明替换为对更简单引理的引用，可以把 sorries 提前到文件前面：

```lean4
-- 改前：
theorem main_theorem : A = C := by sorry

-- 改后：
theorem lemma1 : A = B := by sorry
theorem lemma2 : B = C := by sorry
theorem main_theorem : A = C := by
  rw [lemma1, lemma2]
```

### 证明内部

当证明有多个分支时，先用 `sorry` 填简单分支，**先攻克最难的分支**。如果难分支失败了，简单分支上的工作就白费了。

```lean4
match n with
| 0 => sorry        -- 稍后填
| 1 => sorry        -- 稍后填
| n + 2 =>          -- ← 先在这里工作
```

## Proof Cleanup

证明通过后，**立即清理**：

1. 合并冗余步骤：`rw [a]; rw [b]` → `rw [a, b]`
2. 测试 `simp` 能否处理更多（逐行删除前面的步骤）
3. 找到真正最小的证明
4. 检查 `#check` 确认定理签名正确

**例子：**
```lean4
-- 改前：
example (x : ℕ) : x + 0 = x := by
  rw [add_comm x 0]
  rw [add_zero x]
  rfl

-- 改后：
example (x : ℕ) : x + 0 = x := by
  simp
```

## Dependent Type Rewriting

**问题：** 当重写的项出现在依赖类型中（如 `h : a ≤ b` 中的 `b`）时，`rw` 报错 `motive is not type correct`。

```lean4
have hb : b = f x
rw [hb]  -- Error: motive is not type correct
```

**解法：先推广，后特化。**

先证明关于任意参数的一般化陈述，再实例化：

```lean4
suffices ∀ s, statement_about s by
  have h_specific := the_equality_you_have
  convert this ?_ <;> exact h_specific
intro s
-- 现在证明关于任意 s 的一般化陈述
```

这样 `convert` 在最后处理依赖类型的 coercion。

## Quality Checklist

在声明证明完成前检查：

- [ ] `lake build` 全项目编译通过
- [ ] 零个 `sorry`（在约定范围内）
- [ ] 只有标准公理（`propext`、`Classical.choice`、`Quot.sound`）
- [ ] 没有未经许可的声明修改
- [ ] import 最小化（只引入需要的模块）
- [ ] 证明已清理（无冗余步骤）
- [ ] 大段 `have` 块（>30 行）已提取为独立引理

## Common error patterns and fixes

#### `noncomputable def` breaks definitional equality (`rfl`, `dsimp`)

In Lean 4, `noncomputable def` is compiled as `opaque`, which disables definitional
reduction. This has two consequences:

**1. `rfl` fails on `noncomputable def` equality.** You cannot use `rfl` to prove
`(T F n D).coeff x x = D.coeff x x - c * Id.coeff x x` when `T` is `noncomputable`.
The goal type shows the equality but Lean cannot compute it:

```lean4
-- ❌ rfl fails: T is noncomputable
calc
  (T F n D).coeff x x = ... := rfl

-- ✅ Extract a lemma that states the equality
lemma T_coeff_eq (D : HalfDerivation F n) (x y : MatIdx n) :
    (T F n D).coeff x y = D.coeff x y - scalarCoeff F n D * ((idHalfDeriv F n).coeff x y) :=
  rfl
```

Wait — `rfl` DOES work inside a lemma for `noncomputable def` with `let` binders
(the `let` is definitional even when the enclosing `def` is `noncomputable`). The
ambiguity is real; when in doubt, test `rfl` first. If it fails, use `dsimp [T]` or
`unfold T`.

**2. `let` bindings of `noncomputable` definitions don't reduce with `dsimp`.**

```lean4
-- ❌ dsimp [sys] does nothing when sys := noncomputable_def
let sys := toRecursiveSystem D
have h : D.coeff x x = sys.C x.i x.j := by
  rw [← diagCoeff_eq D x.i x.j ...]
  dsimp [sys, toRecursiveSystem]  -- sys stays folded

-- ✅ Use the noncomputable def directly, no let binder
have h : D.coeff x x = (toRecursiveSystem D).C x.i x.j := by
  rw [← diagCoeff_eq D x.i x.j ...]
  dsimp [toRecursiveSystem]  -- this reduces
```

Rule: when a definition is `noncomputable`, use it directly in expressions
rather than binding it with `let`. The `let` creates an opaque wrapper that
`dsimp` cannot penetrate. Calling `dsimp` on the original `noncomputable def`
works (reduces the structure projection to its definition).

#### `linarith` fails on abstract field `F`

`linarith` works on `ℕ`, `ℤ`, `ℚ`, `ℝ` (ordered commutative semirings). It does **not** work on an arbitrary abstract field `F` (e.g. `F : Type` with `[Field F] [CharNeTwo F]`). The error may just say `linarith failed` with no clear diagnosis — no syntax error, no type error, just a failed tactic.

**Fix:** Use `ring` + `calc` + `simp` for field arithmetic. The go-to pattern for `x = y` when you have `-x - y = 0`:

```lean4
-- Given h_eq : -(a) - b = (0 : F), prove a = -b

-- linarith would be ideal but doesn't work on abstract F
-- Instead, use ring and calc:
have h_sum : a + b = 0 := by
  calc
    a + b = -(-(a) - b) := by ring
    _ = -(0 : F) := by rw [h_eq]
    _ = 0 := by simp
calc
  a = (a + b) - b := by ring
  _ = (0 : F) - b := by rw [h_sum]
  _ = -b := by simp
```

Or when you need `simpa` to apply a `calc` result whose LHS has a different
syntactic form from the goal (e.g., `D.f (n-1) n u n = 0` vs goal
`D.f (n-1) (n-1+1) u n = 0`):

```lean4
have h_result : D.f (n-1) n u n = (0 : F) := by
  calc
    D.f (n-1) n u n = -(-D.f (n-1) n u n) := by ring
    _ = -(0 : F) := by rw [h_neg]
    _ = 0 := by simp
-- Goal has (n-1)+1 instead of n after a subst — bridge with simpa
simpa [show (n-1 : ℕ) + 1 = n by omega] using h_result
```

**General approach:** If the goal is `x = y` and you already have `h_eq : x = y` in context, write `exact h_eq` — do not reach for `linarith` at all. If the equality follows from a case condition, use `apply h_eq` or `rw [h_eq]`. Reserve `linarith` only for goals in `ℕ`/`ℤ`/`ℚ`/`ℝ`.

| Error | Likely cause | Fix |
|-------|-------------|-----|
| `unknown module prefix 'Mathlib'` | LEAN_PATH 不正确 | 用 `lake env lean` 替代直接 `lean` |
| `unknown constant` | 定理名拼错或未 import | `#check` 确认正确名称 |
| `type mismatch` | 类型不匹配 | 检查两边的类型，用 `exact?` 搜索 |
| `tactic 'rewrite' failed` | 重写模式没找到匹配 | 检查等式方向，用 `rw [← h]` |
| `unsolved goals` | 证明不完整 | 看目标还剩什么 |
| `tactic 'ring' failed` | 不是多项式等式或在非交换环上 | 用 `ring_nf` 代替 `ring` |
| `failed to synthesize` | 缺少实例 | import 相关模块 |
| `invalid 'calc' step` | calc 起点与目标不一致 | 先 `simp` 再检查实际目标状态 |
| `No goals to be solved` | 前一个 tactic 已关闭目标 | 将 `rw; simp` 拆成两个 calc 步骤，或移到单独 `have` 中 |
| `omega` can't prove goal with subtraction | `hlen: j-i = d` in context but target is `k-i < d` (doesn't mention `j`) | Don't `rw [hlen]` — `omega` can't use the equality after rewriting. **Fix: use explicit `calc` with `Eq.symm hlen`** to rewrite `d` as `j-i`, then prove `k-i < j-i` using `omega` (or `Nat.sub_lt`): `have h_len : k-i < d := by` `have hd_eq : d = j-i := Eq.symm hlen; rw [hd_eq]; omega` |
| `omega` can't prove `i+1 < j` from `j-i > 1` | `omega` may struggle when the relationship is buried in subtraction | Provide explicit bound: `have hmid : i+1 < j := by have : 1 < j-i := hdgt; omega` |
| `No goals to be solved` after `omega; omega` | `omega` on the first branch closes the goal, second `omega` has nothing to do | Use `omega` once: `by omega` not `by omega; omega`. Use separate `have` lines for intermediate facts, then a single `omega` |
| `noncomputable def` won't reduce with `rfl`/`dsimp`/`simp` | Lean 4 compiles `noncomputable def` as opaque — definitional reduction disabled | Use `unfold` (works when def is in scope) or `rw [myDef]` (uses equation lemma). `rfl` NEVER works on noncomputable defs |
| `≠` direction mismatch: `x.i ≠ i` vs `¬(i = x.i)` | `≠` is `¬ (a = b)` with a specific argument order; binder names in theorem signatures generate different `≠` orientations | Use `Ne.symm h` to flip direction, or `rcases` with `h` and `h.symm` introduced separately |
| `variable (F n)` makes F,n explicit → `T D` fails | Explicit binder requires `T F n D`; Lean reports "Application type mismatch" or treats `D` as Type arg | Add F,n: `T F n D`. Use `variable {F n}` with braces to make implicit if desired |
| `failed to compile definition, consider marking it as 'noncomputable'` | The definition depends on a `noncomputable` function (e.g., `T` uses `scalarCoeff` which has `dite`) | Add `noncomputable` before `def`. This propagates — all downstream `def`s that use `T` must also be `noncomputable`. |
| `omega could not prove the goal` in MatIdx constructor inside a `def` | `def` has no hypotheses constraining `n`, so `1 ≤ n-1` or `2 < n` are unprovable | Add `hn` hypotheses to the `def` signature (e.g., `(hn3 : 3 ≤ n)`). In `theorem` bodies, `omega` works because theorems carry hypotheses. `def`s without `hn` args must handle all `n`, including `n=0` where `n-1 < n` is false. |
| `Function expected at X` / `The identifier X is unknown` | `def X` is defined AFTER the theorem that references it in the same file | Move the `def` before the theorem. Lean processes files top-to-bottom; names are only visible after their definition point. |
| `≠` direction mismatch in function args: `h : x.i ≠ i` but expected `¬(i = x.i)` | `≠` is `¬ (a = b)`, and `a ≠ b` is NOT definitionally `¬ (b = a)` | Use `Ne.symm h` to swap the direction. When the expected type is `¬ (i = x.i)`, pass `Ne.symm hx_not_diag` (where `hx_not_diag : x.i ≠ i`). Same for `i+1 ≠ x.j` → `Ne.symm`. |

This affects calling convention: bracket x y may fail with expected Nat if n is explicit. Fix: change variable (n : Nat) to variable {n : Nat}, or pass (n := n) as a named argument.

Cross-file calls need explicit F n when both files use variable (F n).
When file A defines T under variable (F : Type) ... (n : Nat) and file B
imports it, calls like (T D) fail with Application type mismatch.
The fix: write (T F n D) — explicitly passing all variable parameters
across module boundaries.

## Mathlib Code Style

When writing code intended for mathlib contribution (PR-ready proofs), follow these style conventions. These rules come from the official [leanprover-community style guide](https://leanprover-community.github.io/contribute/style.html).

### Variable Conventions

| Variable | Purpose |
|----------|---------|
| `a`, `b`, `c` | Propositions |
| `α`, `β`, `γ` | Generic types |
| `x`, `y`, `z` | Elements of generic types |
| `h`, `h₁`, `h₂` | Assumptions/hypotheses |
| `p`, `q`, `r` | Predicates and relations |
| `s`, `t` | Lists and sets |
| `m`, `n`, `k` | Natural numbers |
| `i`, `j` | Integers |
| `G` | Group, `R` Ring, `K`/`𝕜` Field, `E` Vector space |

### Indentation & Line Breaking

**`by` at end of preceding line (CRITICAL):**
```lean4
-- ✅ Good: by at end of preceding line
theorem foo : P := by
  exact h

-- ❌ Bad: by alone on its own line
theorem foo : P :=
  by exact h

-- ❌ Bad
theorem foo : P
    := by exact h
```

**Multi-line theorem statements:**
```lean4
-- Short: single line
theorem foo_bar (h : P) : Q := by ...
theorem square_is_ev (n : ℕ) : Ev (n^2) := ...

-- Medium: break after colon, 4-space continuation
theorem foo_bar_with_long_name (h₁ : P) (h₂ : Q) :
    conclusion := by
  ...

-- Long: break at parameters, 4-space continuation
theorem foo_bar_with_very_long_name
    (h₁ : a_very_long_hypothesis_type)
    (h₂ : another_very_long_hypothesis)
    (h₃ : yet_another) : conclusion := by
  ...
```

**One tactic per line** — no `tac1; tac2` chains (exception: sequential `rw` should be **merged** into a single call):
```lean4
-- ❌ Bad: chained with ;
theorem foo : P := by
  subst hk; rfl
  rw [a]; rw [b]

-- ✅ Good: one tactic per line, merge sequential rw
theorem foo : P := by
  subst hk
  rfl
  rw [a, b]    -- merged
  ring
```

**Focused subgoals with `·` (focusing dot):**
```lean4
theorem foo : P ∧ Q := by
  constructor
  · exact hp
  · exact hq

-- <;> to apply same tactic to all goals
theorem bar : P ∧ Q := by
  constructor <;> assumption
```

### `have` Statement Formatting

```lean4
-- Short: single line
have h : P := by exact hp

-- Long justification: by on same line as have
have h : P := by
  apply something
  exact hp
```

### Proof Length & Comments

**Max 50 lines per proof** (target <15 lines for main theorems). Longer proofs should be broken into helper lemmas.

**Mathlib proofs have NO inline comments.** Proofs must be self-documenting through clear variable names and logical structure:

```lean4
-- ❌ Bad: inline comments describing the proof
theorem foo : P := by
  -- First we show that A holds
  have hA : A := by
    -- This is because of lemma bar
    exact bar
  -- Now we can use hA to get B
  have hB : B := trans hA hC
  -- Finally conclude
  exact final_step hB

-- ✅ Good: clean proof, no inline comments
theorem foo : P := by
  have hA : A := bar
  have hB : B := trans hA hC
  exact final_step hB
```

**What to avoid in comments:**
- Play-by-play of each tactic
- "Step N" markers
- Restating what the code obviously does
- Explaining what a tactic does
- Summarizing proof strategy (put in docstring if needed)
- TODO comments (use GitHub issues instead)

**Rarely acceptable** (use sparingly):
- `-- Porting note:` for Lean 3→4 migration issues
- Reference to a paper for a highly unusual technique

### `variable` binder behavior gotcha

`variable (n : ℕ)` makes `n` an **explicit** binder in any definition under it. `variable {n : ℕ}` makes it **implicit**:

```lean4
variable (n : ℕ)  -- explicit binder
def bracket (x y : MatIdx n) : ... := ...
-- bracket n x y  -- n must be passed explicitly

variable {n : ℕ}  -- implicit binder
def bracket (x y : MatIdx n) : ... := ...
-- bracket x y     -- n is inferred from x
```

This affects calling convention: bracket x y may fail with expected Nat if n is explicit. Fix: change variable (n : Nat) to variable {n : Nat}, or pass (n := n) as a named argument.

Cross-file calls need explicit F n when both files use variable (F n).
When file A defines T under variable (F : Type) ... (n : Nat) and file B
imports it, calls like (T D) fail with Application type mismatch.
The fix: write (T F n D) — explicitly passing all variable parameters
across module boundaries.

### `Nat` subtraction lemma signatures

Key `Nat` subtraction lemmas you may need:

**`Nat.sub_lt_sub_right`** `(c ≤ a) → (a < b) → a - c < b - c`
- Use: from `k < j` and `u ≤ k`, get `k - u < j - u`
- Signature: `Nat.sub_lt_sub_right hka hkj` where `hka : u ≤ k` and `hkj : k < j`

**`Nat.sub_lt_sub_left`** `(k < m) → (k < n) → m - n < m - k`
- Use: from `k < j` and `k < v`, get `v - j < v - k`
- Signature: `Nat.sub_lt_sub_left hkv hkj` where `hkv : k < v` and `hkj : k < j`

**`Nat.sub_pos_of_lt`** `(h : a < b) → 0 < b - a` — use for positivity of subtraction.

These all work on truncated ℕ subtraction (where `a - b` = 0 if `a < b`).


**Docstrings describe the STATEMENT, not the proof.** A docstring tells a user browsing the API *what is claimed*, not *how the proof works*.

```lean4
-- ❌ Bad: contains proof strategy
/-- **Sturm bound for level-1 modular forms.** If f has zero coefficient on
q^i for every i ≤ k/12, then f is identically zero. The proof iterates
CuspForm.discriminantEquiv (division by Δ) until the weight goes negative,
where everything is zero. -/
theorem sturm_bound_levelOne ... := ...

-- ✅ Good: statement only, short
/-- **Sturm bound for level-1 modular forms.** If f has zero coefficient on
q^i for every i ≤ k/12, then f is identically zero. -/
theorem sturm_bound_levelOne ... := ...

-- ✅ Good: terse
/-- `f` is continuous. -/
theorem f_continuous : Continuous f := ...
```

Rules:
- Docstrings should be **SHORT** — one sentence stating the claim
- Drop phrases like: "The proof iterates …", "By induction on …", "Uses Lemma X …", "We first establish … then apply …"
- **Only important public theorems** need docstrings (private/auxiliary lemmas must NOT have docstrings)
- Module docstrings (`/-! … -/` at top of file) MAY describe overall proof strategy — only per-lemma docstrings are subject to the statement-not-proof rule

### Syntax Preferences

```lean4
-- ✅ fun over λ (λ is deprecated)
fun x => x + 1

-- ✅ ↦ (mapsto) for lambda abstractions
fun x ↦ x + 1

-- Pattern-match arrows (| zero => 0) still keep =>

-- ✅ <| over $ (not allowed in mathlib)
f <| g x
apply foo <| bar baz

-- ✅ centered dot for simple functions
(· ^ 2)
List.map (· + 1) xs
```

### Inequality Orientation

**Every inequality stated with the smaller side on the left.** Use `≤`, never `≥`. Use `<`, never `>`. Lemma names follow the same convention.

```lean4
-- ❌ Bad: ≥ in statement, name uses _ge_
theorem primeIdealZetaSum_ge_log_minus_bounded (s : ℝ) :
    primeIdealZetaSum K univ s ≥ Real.log (1/(s-1)) - C := ...

-- ✅ Good: ≤ in statement, name uses _le_
theorem log_minus_bounded_le_primeIdealZetaSum (s : ℝ) :
    Real.log (1/(s-1)) - C ≤ primeIdealZetaSum K univ s := ...

-- Same for strict:
-- ❌ lemma f_gt_zero (h : x > 0) : f x > 0 := ...
-- ✅ lemma zero_lt_f (h : 0 < x) : 0 < f x := ...
```

### Calc Block Format

```lean4
calc
  a = b := by ...
  _ = c := by ...
  _ ≤ d := by ...
-- Relations aligned, underscores left-justified
```

### Whitespace Rules

```lean4
-- ✅ Good
a + b * c
def foo (x : α) (y : β) : γ := ...
f x y
(a, b)

-- ❌ Bad: no spaces
a+b*c
def foo (x:α)(y:β):γ := ...

-- ✅ space after ← in rewrites
rw [← add_comm a b]
rw [← h]

-- No trailing whitespace on any line
-- Exactly one blank line between consecutive declarations (not zero, not two)
```

### Line Length

**Maximum**: 100 characters. Fill lines to ~100 characters — do NOT break early.

```lean4
-- ❌ Bad: artificially narrow
  simp only [ne_eq, mul_eq_zero,
    OfNat.ofNat_ne_zero, not_false_eq_true,
    ofReal_eq_zero, Real.pi_ne_zero,
    I_ne_zero, or_self]

-- ✅ Good: fill to ~100 chars
  simp only [ne_eq, mul_eq_zero, OfNat.ofNat_ne_zero, not_false_eq_true, ofReal_eq_zero,
    Real.pi_ne_zero, I_ne_zero, or_self]
```

---

## Proof Golfing

Concrete transformation rules to shorten and clean up Lean proofs. Apply these systematically during cleanup.

### Phase 1: Instant Wins (zero risk)

| # | Before | After | Rule |
|---|--------|-------|------|
| 1.1 | `:= by exact t` | `:= t` | Remove `by exact` wrapper |
| 1.2 | `:= by rfl` | `:= rfl` | Remove `by rfl` |
| 1.3 | `rw [h]; exact e` | `rwa [h]` | `rw + exact` → `rwa` |
| 1.4 | `simp [foo]; exact h` | `simpa [foo] using h` | Consolidate `simp` + `exact` |
| 1.5 | `simp [foo]; rfl` | `simp [foo]` | Remove trailing `rfl` after `simp` |
| 1.6 | `constructor; · exact a; · exact b` | `exact ⟨a, b⟩` | Anonymous constructor |
| 1.7 | `apply f; exact h` | `exact f h` | Collapse `apply` + `exact` |
| 1.8 | `by_contra h; push_neg at h` | `by_contra! h` | Combined `by_contra!` |
| 1.9 | `fun x => f x` | `f` | Eta-reduce lambdas |
| 1.10 | `have h := foo x; exact bar h` | `exact bar (foo x)` | Inline single-use `have` |
| 1.11 | `apply f; intro m` | `refine f fun m => ?_` | `apply + intro` → `refine fun` |
| 1.12 | `show T` when goal is already `T` | *(remove)* | Remove redundant `show` |
| 1.13 | `have h := foo x; ... h.1 ... h.2` | `obtain ⟨a, b⟩ := foo x; ... a ... b` | Destructure early |
| 1.14 | `rw [a]; rw [b]; rw [c]` | `rw [a, b, c]` | Merge consecutive `rw` |
| 1.15 | Terminal `simp only [...]` | `simp` | Unsqueeze terminal `simp` |
| 1.16 | Nonterminal bare `simp` | `simp only [...]` | Squeeze nonterminal `simp` (use `simp?`) |
| 1.17 | `Monotone.comp hf hg` | `hf.comp hg` | Use dot notation |
| 1.18 | `f (by simp)` | `f <| by simp` | Use `<|` to avoid trailing parens |
| 1.19 | `push_neg at h` | `push Not at h` | `push_neg` deprecated |
| 1.20 | Inline trivial single-use `∃`-lemmas | Inline at call site as `obtain ⟨...⟩ : ∃ ... := ⟨...⟩` | Remove junk helper lemmas |

### Phase 2: Automation Upgrades

Try each, keep if it compiles:

**2.1 Try `grind`** on the entire proof body — the most common reviewer suggestion. `grind` subsumes many tactic chains:
```lean4
-- Try replacing the entire proof body with:
  by grind
  by grind [relevant_lemma₁, relevant_lemma₂]
```

**2.2 Try deleting tactics before `grind`** — `grind` often subsumes preceding steps:
```lean4
rw [h]; grind   ↔  grind
simp; grind     ↔  grind
simp [foo] <;> grind  ↔  grind [foo]
```

**2.3 Try `simpa` consolidation** — `simp` followed by `exact h` → `simpa [foo] using h`

**2.4 Try `fun_prop`** — for ANY goal involving `Continuous`, `Differentiable`, `Measurable`, `ContDiff`, etc. Use `fun_prop (disch := grind)` if `fun_prop` alone fails.

**2.5 Try `positivity`** — for goals `0 < x`, `0 ≤ x`, `x ≠ 0` derivable from structure.

**2.6 Try `gcongr`** — for inequality goals in `calc` blocks or goals needing monotonicity.

**2.7 Try `lia` first, then `omega`** — for `Nat`/`Int` arithmetic. Prefer `lia` as the default; only fall back to `omega` if `lia` fails on a specific goal. (`grind` includes `lia` as a module, so the priority order is: `grind` → `lia` → `omega`.)

**2.8 Try `aesop`** — for logic, membership, and simple algebraic structure goals.

**2.9 Try `norm_num` / `norm_cast`** — for concrete numeric computation or cast goals.

**2.10 Try `decide` / `decide +kernel`** — for decidable propositions (finite computation).

**2.11 Try `field_simp; ring`** — for algebraic identities involving denominators.

**2.12 Try `linear_combination`** — for EVERY proof ending in `ring` or `ring_nf` with hypotheses in context.

**2.13 Try `wlog` for symmetric cases** — when two case branches have near-identical proofs.

**2.14 Try collapsing `calc` blocks** — two-step `calc` blocks can often be composed: `calc a ≤ b := h1; _ ≤ c := h2` → `le_trans h1 h2`.

**2.15 Close multiple goals with `<;>`** — when `refine` produces multiple similar goals:
```lean4
refine ⟨?_, ?_, ?_⟩ <;> grind [def, key_fact]
```

### Phase 3: Cleanup (Style & Structure)

| # | Rule | Details |
|---|------|---------|
| 3.1 | `erw` → `rw` | Try `rw` first; only use `erw` when `rw` genuinely fails |
| 3.2 | `continuity` / `measurability` → `fun_prop` | Legacy tactics being replaced |
| 3.3 | `omega` → `lia` | mathlib migrating to `lia`; when `grind` also fails try `lia`, only then `omega` |
| 3.4 | `rcases ... with rfl` auto-substitutes | No need for subsequent `simp [h]` when `rfl` already substituted |
| 3.5 | Register lemmas for automation | Add `@[simp]`, `@[grind]`, `@[fun_prop]`, `@[aesop]` as appropriate |
| 3.6 | `simp_all` over `simp_all only` | For closing goals |
| 3.7 | Remove `set_option maxHeartbeats` | Never acceptable in mathlib — fix the proof instead |

### What doesn't golf well (don't waste time)
- Analysis boilerplate (integrability/measurability chains) — needs new API
- `grind` with `zpow`, `Even`/`Odd`, or cast arithmetic — known limitation
- Dominated convergence sub-proofs — already near-minimal
- `grind` in large contexts — can timeout; keep standalone lemmas

**Minimum value filter:** 1-line savings are only worth making if zero-risk syntax cleanup (e.g., `by exact` → term) OR they also improve clarity or performance. Don't churn code for marginal compression.

---

## Audit methodology: verifying Lean structures against standard definitions

When Lean produces a result that seems to contradict known mathematics, follow
this audit protocol:

### 1. Read the Lean definition

Compare the Lean structure field against the mathematical definition item by
item. Trace the derivation: apply the standard definition to specific basis
elements, expand both sides, and verify each term in the Lean condition.

### 2. Construct the counterexample in Lean

If analysis suggests a counterexample, construct it directly in the Lean
structure and let Lean verify. Adjacent-source cases may be vacuous (no k-split
exists), making verification straightforward.

### 3. Derive Lean's condition from the standard definition

Pick specific basis elements where the bracket is non-zero. For the
half-derivation condition on N_n: set x₁=E(i,k), x₂=E(k,j) so [x₁,x₂]=E(i,j).
Expand 2·D([x₁,x₂]) = [D(x₁),x₂] + [x₁, D(x₂)] coefficient by coefficient. Each
non-zero bracket case corresponds to one RHS term in Lean's formula.

### 4. Trace the paper's argument chain

Map the paper's argument as a dependency graph. Check each dependency's scope:
which source types it covers. Common gaps: implied completeness (hardest case
silently omitted), type confusion (adjacent vs nonadjacent), domain shift
(lemma proven for one type assumed for another).

### 5. Find the structural explanation

When a counterexample exists, find the algebraic structure explaining it.
For 1/2-derivations on N_n: Hom(L/[L,L], Z(L)) explains extra dimensions as
maps from the abelianization to the center.

---

## References

- **Official**: Theorem Proving in Lean 4 — https://lean-lang.org/theorem_proving_in_lean4/
- **Practice**: Mathematics in Lean — https://leanprover-community.github.io/mathematics_in_lean/
- **Game**: Natural Number Game — https://adam.math.hhu.de/#/g/leanprover-community/NNG4
- **Cheat Sheet**: https://lean4.dev/tactics/cheat-sheet
- **PDF Cheatsheet**: https://github.com/madvorak/lean4-cheatsheet
- **Community AI Skills**: https://github.com/cameronfreer/lean4-skills
- **Mathlib Code Style**: https://leanprover-community.github.io/contribute/style.html
- **Mathlib Quality Skill**: https://github.com/CBirkbeck/mathlib-quality
- **Mathlib Theorem Quick Reference**: See `references/mathlib-theorem-quickref.md`
- **Verified Import Paths**: See `references/verified-import-paths.md`
- **Zero-Bracket Encoding (cond_zero)**: See `references/zero-bracket-encoding.md`
- **coeffOf_cond Bridge (dite→ite)**: See `references/coeffOf-cond-bridge.md`
- **cond_zero Bracket Expansion Gap**: See `references/cond-zero-bracket-gap.md`
- **Disjoint-Target Limitation**: See `references/disjoint-target-limitation.md`
- **Central Perturbation / Corner Coefficient**: See `references/central-perturbation-discovery.md`
- **Structural Classification Methodology**: See `references/structural-classification-methodology.md`
- **verify-build.lean**: Quick build verification script (12 paths, 8 domains)
- **audit-skill-claims.lean**: Full skill audit script (35 tactics, theorem names, patterns)
