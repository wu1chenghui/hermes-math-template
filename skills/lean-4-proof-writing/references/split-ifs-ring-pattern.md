# split_ifs + ring: Linear Condition Proofs

## When to use

When proving that a linear combination satisfies a structure field condition
(e.g., `half_deriv_cond` for `T = D - c·Id`), where the RHS involves
`dite`/`ite` terms. The `split_ifs` + `ring` pattern is more robust than
`h_if_distrib` + `rw` chains.

## The pattern

```lean4
-- Proving T.half_deriv_cond for T = D - c·Id:
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

## Why it works

`split_ifs` expands ALL `dite`/`ite` terms in the goal simultaneously.
Each case replaces every conditional by either its `then` or `else` branch,
leaving a pure algebraic (polynomial) identity that `ring` can close.

The 16 cases arise from 4 independent `dite` conditions (each true or false).
All cases are solved uniformly with `ring` — no case-specific reasoning.

## Why the alternative fails

The `h_if_distrib` approach:

```lean4
have h_if_distrib (c : Prop) [Decidable c] {a b : MatIdx n} :
    (if c then D.coeff a b - h_sc * Id.coeff a b else 0)
    = (if c then D.coeff a b else 0) - h_sc * (if c then Id.coeff a b else 0) := by
  split <;> simp
...
repeat (rw [h_if_distrib])
ring
```

This often FAILS because:
1. `rw` can't match the `dite` binder `h` inside complex MatIdx constructor terms
2. `ring` can't handle the expanded `dite` binders with proof terms embedded
3. The `rw` + `ring` chain leaves unsolved goals when the LHS/RHS have
   `h_sc *` distributed differently

## Concrete example from DerivedZero.lean

The `T` definition in the project used this pattern successfully:

```lean4
noncomputable def T (D : HalfDerivation F n) : HalfDerivation F n :=
  let Tcoeff : MatIdx n → MatIdx n → F := λ x y =>
    D.coeff x y - scalarCoeff F n D * ((idHalfDeriv F n).coeff x y)
  haveI : CharNeTwo F := inferInstance
  have h_half_deriv := by
    intro x y k hk hk'
    dsimp [Tcoeff]
    have hD := D.half_deriv_cond x y k hk hk'
    have hId := (idHalfDeriv F n).half_deriv_cond x y k hk hk'
    have h_expand : (2 : F) * (D.coeff x y - scalarCoeff F n D * ((idHalfDeriv F n).coeff x y))
        = ((2 : F) * D.coeff x y) - scalarCoeff F n D * ((2 : F) * (idHalfDeriv F n).coeff x y) := by ring
    rw [h_expand, hD, hId]
    split_ifs
    · ring  -- repeated 16 times
    ...
  { coeff := Tcoeff
    half_deriv_cond := h_half_deriv
  }
```

The `split_ifs` creates 16 subgoals (4 dite conditions: `y.j = x.j ∧ y.i < k`,
`y.i = k ∧ x.j < y.j`, `y.j = k ∧ y.i < x.i`, `y.i = x.i ∧ k < y.j`).
Each subgoal replaces all conditions with their truth values, and `ring`
closes the resulting polynomial identity.
