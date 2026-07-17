# Definitional Equality and Implicit Binders

## The trap

`rfl` and `simp` behave differently depending on whether typeclass variables
(`F`, `n`, etc.) are **explicit** (`variable (F n)`) or **implicit**
(`variable {F n}`).  Code that compiles in `lean_run_code` isolation can fail
in the file context, and vice versa.

**`lean_run_code` is NOT a faithful replica of file compilation.**  Do not
trust a `lean_run_code` success as evidence the same proof will work in-file.

## Root cause

When `variable {F n}` makes binders implicit, the Lean kernel sometimes cannot
definitionally reduce structure-instance field projections like
`(D1 + D2).coeff` even though the `Add` instance defines `coeff` pointwise as
`D1.coeff + D2.coeff`.  With explicit `variable (F n)` the same reduction
succeeds.

Consequences:
- `rfl` fails on goals like `(D1 + D2).coeff i j u v = D1.coeff i j u v + D2.coeff i j u v`
- `dsimp` makes no progress on the same goals
- `simp` works in `lean_run_code` but "makes no progress" in the file

## The fix: bridge `@[simp]` lemmas at the instance definition site

Add `@[simp]` helper lemmas in the file where the instances are defined
(where `variable (F n)` are explicit, so `rfl` works):

```lean
variable (F n)

instance : Add (HalfDerivation F n) where
  add D1 D2 := { coeff := λ i j u v => D1.coeff i j u v + D2.coeff i j u v; ... }

@[simp] lemma coeff_add (D1 D2 : HalfDerivation F n) (i j u v : ℕ) :
    (D1 + D2).coeff i j u v = D1.coeff i j u v + D2.coeff i j u v := rfl

@[simp] lemma coeff_smul (c : F) (D : HalfDerivation F n) (i j u v : ℕ) :
    (c • D).coeff i j u v = c * D.coeff i j u v := rfl

@[simp] lemma coeff_neg (D : HalfDerivation F n) (i j u v : ℕ) :
    (-D).coeff i j u v = (-1 : F) * D.coeff i j u v := rfl
```

Downstream files (with `variable {F n}`) then use `simp` which picks up these
lemmas — `simp` rewrites via `@[simp]` lemmas, not definitional reduction.

### `Sub` is special

`Sub D1 D2 := D1 + (-D2)`.  Therefore `(D1 - D2).coeff` reduces NOT to
`D1.coeff - D2.coeff` but to `D1.coeff + (-1)*D2.coeff`.  The field-level
`-x` is the primitive `Neg.neg` while `(-1)*x` is `Mul.mul` — different
operations, not definitionally equal.

Fix with `calc` using the other bridge lemmas:
```lean
@[simp] lemma coeff_sub (D1 D2 : HalfDerivation F n) (i j u v : ℕ) :
    (D1 - D2).coeff i j u v = D1.coeff i j u v - D2.coeff i j u v := by
  calc
    (D1 - D2).coeff i j u v = (D1 + (-D2)).coeff i j u v := rfl
    _ = D1.coeff i j u v + (-D2).coeff i j u v := by rw [coeff_add]
    _ = D1.coeff i j u v + ((-1 : F) * D2.coeff i j u v) := rfl
    _ = D1.coeff i j u v - D2.coeff i j u v := by ring
```

The third step uses `rfl` (NOT `coeff_neg` — that lemma may be defined later,
causing a forward-reference error).  In the instance-definition file with
explicit binders, `(-D).coeff` IS definitionally `(-1)*D.coeff`.

## Related: `dite` and `simp` with `by_cases` hypotheses

`simp` does NOT automatically use hypotheses introduced by `by_cases` to
simplify `dite` (dependent `if`) conditions.  Always pass them explicitly:

```lean
-- WRONG:
unfold coeffOf; simp

-- RIGHT:
unfold coeffOf; simp [hi, hij, hjn, hu, huv, hvn]
```

This pattern works for all 6 validity guards of `coeffOf`.

## The `coeffOf_f` pattern

When index validity is known, bridge from `coeffOf` to raw `coeff`:

```lean
rw [coeffOf_f D i j u v hi hij hjn hu huv hvn]  -- strips all if-guards at once
-- goal now involves D.coeff, where bridge lemmas apply
simp  -- picks up coeff_add/coeff_smul/coeff_neg
```

This is preferred over `unfold coeffOf; split_ifs` because `split_ifs`
behavior also differs between `lean_run_code` and file context.
