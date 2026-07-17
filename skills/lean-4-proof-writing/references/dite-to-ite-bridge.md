# Dite-to-Ite Bridge Pattern: HalfDerivation → NCoeff

## Problem

You have a **dependent-type source** structure (e.g., `HalfDerivation`) whose field
`half_deriv_cond` uses `dite` (dependent if) with MatIdx proof terms baked into the
branch values:

```lean4
half_deriv_cond (x y : MatIdx n) (k : ℕ) (hpos : x.i < k) (hlt : k < x.j) :
    (2 : F) * coeff x y =
    (dite (h : x.j = y.j ∧ y.i < k) (λ h => coeff (x.left k hpos hlt) (y.left k h.2)) (λ _ => 0))
  - (dite (h : y.i = k ∧ x.j < y.j) ... )
  + (dite (h : x.i = y.i ∧ y.i < k) ... )
  - (dite (h : y.j = k ∧ y.i < x.i) ... )
```

Here:
- `h.2 : y.i < k` is used as a **MatIdx constructor argument** (`y.left k ... h.2`)
- The `dite` binder names (`h`) are part of the proof term
- The MatIdx `hlt` field is a propositional proof that differs each time the condition holds

You need to prove equality with a **plain-indexed target** (e.g., `NCoeff`) whose `cond`
field uses `ite` (ordinary if) with independent ℕ-indexed proofs:

```lean4
cond i j u v ... k ... :
    (2 : F) * f i j u v =
    (if v = j ∧ u < k then f i k u k else 0)
  - (if u = k ∧ j < v then f k j v k else 0)
  + (if u = i ∧ k < v then f k j k v else 0)
  - (if v = k ∧ u < i then f i k u i else 0)
```

The `dite` and `ite` represent the **same mathematical condition**, but differ in:
1. `dite` vs `ite` syntax
2. Dependent proof terms (MatIdx `hlt`) vs independent indices
3. Named binder `h` vs condition expressions

## Solution: `coeffOf` + external lemma pattern

### Step 1: Define `coeffOf` as a standalone def

Define the coefficient function BEFORE the `structure where` block so it's
a plain `def` that `unfold` and `simp` can access:

```lean4
noncomputable def coeffOf (D : HalfDerivation F n) (i j u v : ℕ) : F :=
  if hi : 1 ≤ i then
    if hij : i < j then
      if hjn : j ≤ n then
        if hu : 1 ≤ u then
          if huv : u < v then
            if hvn : v ≤ n then
              D.coeff (⟨i, j, hi, hij, hjn⟩) (⟨u, v, hu, huv, hvn⟩)
            else 0
          else 0
        else 0
      else 0
    else 0
  else 0
```

All 6 nested `if`s handle out-of-bounds indices by returning `0`. Only when all
bounds hold do we actually look up `D.coeff`.

### Step 2: Write a `coeffOf_f` wrapper lemma

This lets callers rewrite `coeffOf D i j u v` to the actual `D.coeff` call when
all bounds are known:

```lean4
lemma coeffOf_f (D : HalfDerivation F n) (i j u v : ℕ)
    (hi : 1 ≤ i) (hij : i < j) (hjn : j ≤ n)
    (hu : 1 ≤ u) (huv : u < v) (hvn : v ≤ n) :
    coeffOf D i j u v = D.coeff (⟨i, j, hi, hij, hjn⟩) (⟨u, v, hu, huv, hvn⟩) := by
  unfold coeffOf
  simp [hi, hij, hjn, hu, huv, hvn]
```

`simp` with all 6 proof hypotheses resolves every nested `if` to the true branch.

### Step 3: The `cond` bridge lemma with 16-case pattern

```lean4
lemma coeffOf_cond (D : HalfDerivation F n) (i j u v : ℕ)
    (hi : 1 ≤ i) (hij : i < j) (hjn : j ≤ n)
    (hu : 1 ≤ u) (huv : u < v) (hvn : v ≤ n)
    (k : ℕ) (hk : i < k) (hk' : k < j) :
    (2 : F) * coeffOf D i j u v =
    (if u = k ∧ j < v then coeffOf D k j k v else 0)
  + (if v = j ∧ u < k then coeffOf D i k u k else 0)
  - (if u = i ∧ k < v then coeffOf D k j k v else 0)
  - (if v = k ∧ u < i then coeffOf D i k u i else 0) := by
  unfold coeffOf
  simp [hi, hij, hjn, hu, huv, hvn]
  -- Goal: (2:F)*D.coeff x y = (dite ...) - (dite ...) + (dite ...) - (dite ...)
  -- where dites use h.1/h.2 as proof terms for MatIdx
  -- and the target ite-expression is exactly the pattern we need
  have h_cond := D.half_deriv_cond (⟨i, j, hi, hij, hjn⟩) (⟨u, v, hu, huv, hvn⟩) k hk hk'
  -- h_cond has dites with h.1, h.2 proof terms
  -- The goal has ites of the form: (if condition then f i k u k else 0)
  -- We need to show both sides are equal term-by-term

  -- Initialize with the identity from h_cond
  calc
    (2 : F) * D.coeff (⟨i, j, hi, hij, hjn⟩) (⟨u, v, hu, huv, hvn⟩) = ... := h_cond
    _ = ... := ...

  -- For each of the 4 terms, we need to match:
  -- dite (h : v=j ∧ u<k) (λ h => D.coeff (left ... h.1?) (...)) 0
  -- with: ite (v=j ∧ u<k) (D.coeff ...) 0
  -- Use by_cases on each condition independently — see Step 4
```

### Step 4: The 16-case `by_cases` + `simp` pattern

The key insight: `simp` with condition hypotheses resolves **both** the `dite` AND
the `ite` to the same concrete value, because:

- `dite (h : P)` with `simp [hP]` reduces to the then-branch, where `h.1` and `h.2`
  are replaced by the concrete proofs
- `ite P` with `simp [hP]` reduces to the same then-branch

```lean4
  -- Work on each term pair independently
  calc
    (2 : F) * D.coeff ... ... =
      ((dite (h : v = j ∧ u < k) (λ h => D.coeff (⟨...⟩) (⟨...⟩)) (λ _ => 0))
    - (dite (h : u = k ∧ j < v) (λ h => D.coeff (⟨...⟩) (⟨...⟩)) (λ _ => 0))
    + (dite (h : u = i ∧ k < v) (λ h => D.coeff (⟨...⟩) (⟨...⟩)) (λ _ => 0))
    - (dite (h : v = k ∧ u < i) (λ h => D.coeff (⟨...⟩) (⟨...⟩)) (λ _ => 0))) := h_cond

    _ = ((if v = j ∧ u < k then ... else 0)
        - (if u = k ∧ j < v then ... else 0)
        + (if u = i ∧ k < v then ... else 0)
        - (if v = k ∧ u < i then ... else 0)) := by
      -- Show each dite = corresponding ite
      have h_eq1 : (dite (h : v = j ∧ u < k) ...) = (if v = j ∧ u < k then D.coeff ... ... else 0) := by
        -- 16-case by_cases pattern:
        by_cases h1 : v = j ∧ u < k
        · rcases h1 with ⟨hvj, huk⟩
          simp [hvj, huk, coeffOf, hi, hij, hjn, hu, huv, hvn, hk, hk']
        · simp [h1]
      ...
      -- Apply all 4 equality lemmas
      simp [h_eq1, h_eq2, h_eq3, h_eq4]
```

#### Why the 16-case pattern works

The `dite (h : v=j ∧ u<k)` creates a binder `h` where:
- `h.1 : v = j` (proof of first conjunct)
- `h.2 : u < k` (proof of second conjunct, **also** used as `hlt` proof in MatIdx)

`simp [hvj, huk]`:
1. Replaces `h.1` with `hvj` and `h.2` with `huk` in the `dite` body via `h` binder
2. Resolves `dite` to the then-branch via `h1` being true
3. Resolves `ite` to the then-branch via the same condition

After `simp [hvj, huk]`, both sides become identical `D.coeff` terms.

#### MatIdx ext_iff and proof irrelevance

The one remaining issue: even after `simp`, the `D.coeff` calls may differ in their
MatIdx proof terms (different proofs of `u < k`, `k ≤ n`, etc.):

```lean4
-- Left side (from dite): D.coeff ⟨i, k, hi, h1⟩ ⟨u, k, hu, h2⟩
--   where h1: i < k (hk from context) and h2: u < k (from h.2)
-- Right side (from ite):   D.coeff ⟨i, k, hi, hk⟩ ⟨u, k, hu, huk⟩
--   where hk (from function params) and huk (from by_cases)
```

Lean's `coeff` function is `∀ {x y : MatIdx n}, F` — it depends on `MatIdx` values,
not their internal proof terms. Since `MatIdx` is a structure, two instances with
the same `i`, `j` fields are propositionally equal:

```lean4
-- Option A: MatIdx.ext_iff lemma
lemma MatIdx.ext_iff {n : ℕ} (a b : MatIdx n) : a = b ↔ a.i = b.i ∧ a.j = b.j := by
  constructor
  · intro h; subst h; simp
  · intro ⟨hi, hj⟩
    rcases hi with rfl
    rcases hj with rfl
    rfl  -- proof fields are propositional, so indistinguishable

-- Then: have h_eq_mat : ⟨i,k,hi,hk,hkn⟩ = ⟨i,k,hi,hk,hkn⟩ := rfl (same!)
-- But for different proofs: the ext lemma lets you show equality
```

**If the `D.coeff` is a projection of a structure** (e.g., `NCoeff.f`), MatIdx
equality is handled automatically — `simp` and `rfl` treat propositional proof
fields as irrelevant within the structure equality.

**If `D.coeff` is a standalone function** taking MatIdx as explicit arguments,
you may need `apply congrArg D.coeff; ext; simp` to prove equality of MatIdx
instances with different proofs.

### Step 5: Assemble in the `structure where` block

```lean4
noncomputable def toNCoeff (D : HalfDerivation F n) : NCoeff F n where
  f := coeffOf D
  cond := coeffOf_cond D
  cond_zero := coeffOf_cond_zero D
```

Since `coeffOf_cond` and `coeffOf_cond_zero` are external lemmas that operate
on `coeffOf D i j u v` directly (not on `(toNCoeff D).f`), they compile fine.
Inside the `where` block, `f := coeffOf D` is a let-style definition, but the
`cond` field just calls the external lemma — no `unfold f` needed.

## Key insight: what makes this work

1. **`coeffOf` is a plain `def`, not a structure field.** `unfold` and `simp`
   work on it. Inside a `structure where`, the field `f` is NOT unfoldable.

2. **The `by_cases` pattern resolves both `dite` and `ite` simultaneously.**
   `simp` with the condition hypotheses reduces both to the same concrete value.
   The mathematical equality is trivial — it's just about the encoding difference.

3. **MatIdx proof terms are propositional.** Two `MatIdx` instances with the
   same `i` and `j` are equal regardless of how their `<`/`≤` proofs look.
   Lean handles this via `rfl` in most cases, or `MatIdx.ext_iff` in edge cases.

## Common pitfalls

### `dite` binder names collide with `by_cases` names

When `h : v=j ∧ u<k` in `dite` and `h1 : v=j ∧ u<k` in `by_cases`, they're
independent — don't confuse them. Inside the `simp` block after `rcases h1`,
the `dite`'s binder `h` is in the term while `h1` is a hypothesis. `simp` uses
both: `h` via `h1` condition, `h1` via its components `hvj`/`huk`.

### Leftover `h.2` reference in MatIdx after `dsimp`

After `simp [hvj, huk]`, you might see:

```
D.coeff (⟨i, k, hi, hk, ...⟩) (⟨u, k, hu, h.2, ...⟩) = D.coeff (⟨i, k, hi, hk, ...⟩) (⟨u, k, hu, huk, ...⟩)
```

The `h.2` is from the `dite` binder — `simp` rewrote the `dite` body but `h`
is still alive inside the MatIdx term. Fix: `dsimp` to remove the binder
reference, then `rfl`:

```lean4
by_cases h1 : v = j ∧ u < k
· rcases h1 with ⟨hvj, huk⟩
  simp [hvj, huk, coeffOf, hi, hij, hjn, hu, huv, hvn, hk, hk']
  -- If h.2 still appears, add: dsimp; rfl
  dsimp
  rfl
· simp [h1, coeffOf, hi, hij, hjn, hu, huv, hvn, hk, hk']
```

### `subst` + `open MatIdx` → `i` resolves to projection

Inside a `cond_zero` or `cond` bridge where `open MatIdx` is active, `subst h`
can make `i` resolve to the `MatIdx.i` projection function instead of the binder
`i : ℕ`. The error message is: `don't know how to synthesize implicit argument 'α'`.

**Fix:** Use `subst u` (substituting the OTHER variable by name) or `rw [h]`
instead of `subst h`.

### `2≠0` requirement

The NCoeff structure's `cond` provides `(2:F)*f ... = ...`. To conclude `f ... = 0`
from `(2:F)*f ... = 0`, you need `(2:F) ≠ 0`. This is typically a parameter of
the containing theorem:

```lean4
theorem diagonalization (D : NCoeff F n) (h2ne : (2 : F) ≠ 0) : ... := ...
```

In the bridge lemma, this isn't needed — you just prove the `(2:F)*f ... = ...`
identity. The caller uses `h2ne` to divide by 2.

## See also

- SKILL.md: "`structure where` block: field f is NOT unfoldable" section
- SKILL.md: "The 16-case `by_cases` pattern for dite/ite conversion" section
- SKILL.md: "The `subst` binder-confusion bug" section
- `references/half-derivation-bridge.md` — a different bridge pattern (diagonal-only)
