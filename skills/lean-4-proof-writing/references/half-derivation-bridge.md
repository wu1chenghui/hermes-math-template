# Bridge Lemma Pattern: HalfDerivation → RecursiveSystem

## Problem

You have a structure encoding a mathematical object (e.g., a 1/2-derivation `D : 𝔫ₙ → 𝔫ₙ`) via dependent types (`MatIdx n`) and need to extract a simpler index-based structure (`RecursiveSystem F n` with `C : ℕ → ℕ → F`).

The challenge: the target structure expects indexed coefficients `C i j` for ALL `i<j≤n`, but the source structure only provides coefficients through a dependent lookup `D.coeff (⟨i,j,...⟩) (⟨i,j,...⟩)` that requires proofs `hi: 1≤i`, `hij: i<j`, `hjn: j≤n` as part of the key.

## Solution: conditional definition + wrapper lemma

### Step 1: Define a conditional coefficient function

```lean4
noncomputable def diagCoeff (D : HalfDerivation F n) (i j : ℕ) : F :=
  if hi : 1 ≤ i then
    if hij : i < j then
      if hjn : j ≤ n then
        D.coeff (⟨i, j, hi, hij, hjn⟩) (⟨i, j, hi, hij, hjn⟩)
      else 0
    else 0
  else 0
```

This uses nested `if` to handle out-of-bounds indices by returning `0`. The `noncomputable` marker is needed because `HalfDerivation` is a `noncomputable` structure (depends on `MatIdx` which depends on `ℕ`).

### Step 2: Write a wrapper lemma for the valid case

```lean4
lemma diagCoeff_eq (D : HalfDerivation F n) (i j : ℕ) (hi : 1 ≤ i) (hij : i < j) (hjn : j ≤ n) :
    diagCoeff D i j = D.coeff (⟨i, j, hi, hij, hjn⟩) (⟨i, j, hi, hij, hjn⟩) := by
  unfold diagCoeff
  simp [hi, hij, hjn]
```

`simp` with the three proof hypotheses simplifies each nested `if` to its true branch.

### Step 3: Use `half_deriv_cond` to prove the recursion relation

```lean4
lemma recursion_holds (D : HalfDerivation F n) (i j k : ℕ)
    (hi : 1 ≤ i) (hik : i < k) (hkj : k < j) (hjn : j ≤ n) :
    (2 : F) * diagCoeff D i j = diagCoeff D i k + diagCoeff D k j := by
  -- Build MatIdx elements for the three positions
  let x : MatIdx n := ⟨i, j, hi, hik_trans, hjn⟩
  let x_ik : MatIdx n := ⟨i, k, hi, hik, hkj_le_n⟩
  let x_kj : MatIdx n := ⟨k, j, le_trans hi (Nat.le_of_lt hik), hkj, hjn⟩

  -- Apply half_deriv_cond with y = x (diagonal case)
  have h_cond := D.half_deriv_cond x x k hk_gt_xi hk_lt_xj

  -- Simplify the 4 if-then-else terms using known equalities/inequalities
  have h_simplified : (2 : F) * D.coeff x x = D.coeff x_ik x_ik + D.coeff x_kj x_kj := by
    simpa [show x.j = x.j from rfl, show x.i ≠ k from by omega, show x.j ≠ k from by omega,
      show x.i = x.i from rfl, hik, hkj] using h_cond

  -- Show equality of MatIdx structures using i,j field equality
  have h_left_eq : (x.left k hk_gt_xi hk_lt_xj) = x_ik := by
    apply (MatIdx.ext_iff _ _).mpr; simp [x, x.left, x_ik]
  -- ... similar lemmas for other terms ...

  have h_eq : (2 : F) * D.coeff x x = D.coeff x_ik x_ik + D.coeff x_kj x_kj := by
    rw [h_left_eq, ...] at h_simplified; exact h_simplified

  -- Finally convert diagCoeff to coeff
  unfold diagCoeff
  simp [hi, hik_trans, hjn, hik, hkj_le_n, le_trans hi (Nat.le_of_lt hik), hkj]
  exact h_eq
```

### MatIdx equality: `ext_iff` pattern (when `MatIdx.ext` is missing)

`MatIdx` is a structure with fields `(i j : ℕ)` and proof fields `(hpos : 1 ≤ i) (hlt : i < j) (hle : j ≤ n)`. By default, `MatIdx.ext` does NOT exist. You need to define a custom equivalence:

```lean4
lemma MatIdx.ext_iff {n : ℕ} (a b : MatIdx n) : a = b ↔ a.i = b.i ∧ a.j = b.j := by
  constructor
  · intro h; subst h; simp
  · intro h
    rcases h with ⟨hi, hj⟩
    cases a; cases b
    constructor
    · simpa using hi   -- hi was a.i = b.i, now becomes i✝¹ = i✝
    · simpa using hj   -- similar for j
```

**Critical `cases` pattern:** After `cases a; cases b`, the `hi` and `hj` hypotheses still refer to the ORIGINAL `a,b` via field projections, not the new binder names. Use `simpa` (which simplifies the projections) rather than trying to `exact hi` directly.

### Step 4: Construct the target structure

```lean4
noncomputable def toRecursiveSystem (D : HalfDerivation F n) : RecursiveSystem F n where
  C := diagCoeff D
  recursion := by
    intro i j k hi hik hkj hjn
    exact recursion_holds D i j k hi hik hkj hjn
```

## Key insight: recursion doesn't need diagonalization

The recursion relation `2·C(i,j) = C(i,k) + C(k,j)` follows directly from `half_deriv_cond` on the DIAGONAL case (`y = x = E_{ij}`). The 4 if-then-else terms simplify to:

- **Term IA** (v = j ∧ u < k): `coeff(E_{ik}, E_{ik}) = C(i,k)` ✓
- **Term IB** (u = k ∧ j < v): `u = k` is `i = k`, which is FALSE (since `i < k`) → 0 ✓
- **Term IIA** (v = k ∧ u < i): `v = k` is `j = k`, which is FALSE (since `k < j`) → 0 ✓
- **Term IIB** (u = i ∧ k < v): `coeff(E_{kj}, E_{kj}) = C(k,j)` ✓

This holds for ANY half-derivation — diagonal or not — because the formula only involves OTHER diagonal coefficients.

## Pitfalls

### ⚠️ `variable` binder makes F, n explicit parameters

If `diagCoeff` is defined under `variable (F : Type) [Field F] (n : ℕ)`, then `F` and `n` become EXPLICIT parameters of `diagCoeff`:

```lean4
-- Signature (F and n are explicit):
diagCoeff (F : Type) [Field F] (n : ℕ) (D : HalfDerivation F n) (i j : ℕ) : F

-- Calling it:
diagCoeff F n D i j     -- ✅ Must pass F and n explicitly
-- D.diagCoeff i j      -- ❌ ERROR: D is treated as F (Type parameter)
```

**Fix options:**
1. Define `diagCoeff` OUTSIDE the `HalfDerivation` namespace at the top level, WITHOUT a surrounding `variable` binder. Use explicit binder parameters:
   ```lean4
   noncomputable def diagCoeff {F : Type} [Field F] [CharNeTwo F] {n : ℕ}
       (D : HalfDerivation F n) (i j : ℕ) : F := ...
   ```
   The `{}` makes F,n implicit, inferred from D.
2. Keep inside namespace, use `diagCoeff (F := F) (n := n) D i j` when calling.

The `unfold` tactic works correctly regardless: `unfold diagCoeff` expands the definition.

### ⚠️ `let` binder doesn't unfold in `simp`

`let x := ...` creates a local definition that `simp` does NOT unfold. To expand `x` use `dsimp`:

```lean4
let x : MatIdx n := ⟨i, j, hi, hik_trans, hjn⟩

-- ❌ simpa [x, hik] ...  -- x stays folded, hik can't match x.i < k
-- ✅ dsimp [x] at *; simp [hik]  -- x is expanded, now hik works
-- ✅ dsimp [x]; simp [hik]       -- same, but only in the goal

-- For proving equality of structure instances:
have h_eq : x.right k ... = x_kj := by
  dsimp [x, x_kj, MatIdx.right]  -- unfold both sides
  rfl
```

Without `dsimp`, `x.right` stays as `MatIdx.right (let x := ...)` which `simp` can't reduce.

### ⚠️ `simpa` with hypotheses as rewrite rules

`simp` and `simpa` CANNOT use lemmas with hypotheses as rewrite rules:

```lean4
lemma lemmawithHyp (h : 1 ≤ i) : foo = bar := ...

-- ❌ simpa [lemmawithHyp h] ...  -- Error: lemmawithHyp is not a simp lemma
-- ✅ unfold diagCoeff; simp [h] ...  -- Direct approach
```

Instead of passing parameterized lemmas to `simpa`, UNFOLD the definition and `simp` with the hypotheses directly.

### ⚠️ `rw` vs `subst` for variable equality

See the SKILL.md section on `The subst binder-confusion bug`.

### ⚠️ `simpa [hif]` for `if` simplification

When simplifying `(if hik' : x.i < k then ... else 0)` where `hik : i < k` but the condition uses `x.i` (a `let` binder that hides `i`):

```lean4
-- ❌ simpa [hik] ...  -- x.i is not i until dsimp unfolds it
-- ✅ dsimp [x]; simpa [hik] ...  -- now x.i becomes i, and hik applies
-- ✅ simpa [x, hik] ...           -- x in the simp set triggers dsimp
```

Rule: `x` in the `simp` arguments triggers `dsimp` for `x`, but `x` in the `dsimp` command is more reliable.
