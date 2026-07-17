# coeffOf_cond Proof Pattern: split_ifs + ring

## Problem

Bridge `HalfDerivation.half_deriv_cond` (MatIdx + dite) to
`NCoeff.cond` (ℕ + ite). The dite terms have 4 independent conditions
giving up to 16 cases.

## Solution: split_ifs + ring

Instead of the `calc` + `h_if_distrib` + `repeat rw` approach (which
fails because `ring` can't handle dite binders), use `split_ifs` to
expand all conditions into cases, then `ring` in each case.

```lean4
lemma coeffOf_cond (D : HalfDerivation F n)
    (i j u v : ℕ) (hi : 1 ≤ i) (hij : i < j) (hjn : j ≤ n)
    (hu : 1 ≤ u) (huv : u < v) (hvn : v ≤ n)
    (k : ℕ) (hk : i < k) (hk' : k < j) :
    (2 : F) * coeffOf D i j u v = ... := by
  let x : MatIdx n := ⟨i, j, hi, hij, hjn⟩
  let y : MatIdx n := ⟨u, v, hu, huv, hvn⟩
  -- Extract h_cond from half_deriv_cond
  have h_cond := D.half_deriv_cond x y k (by dsimp [x]; exact hk) (by dsimp [x]; exact hk')
  -- Rewrite LHS via coeffOf_f, then RHS via h_cond
  rw [(coeffOf_f D i j u v hi hij hjn hu huv hvn).symm, h_cond]
  -- Expand all MatIdx constructors
  dsimp [x, y, MatIdx.left, MatIdx.right]
  -- Now: dite_RHS = ite_RHS (4 dite terms on LHS, 4 ite terms on RHS)
  -- Use split_ifs on each term individually
  have h_IA : (if h : v=j ∧ u<k then ... else 0) = (if v=j ∧ u<k then coeffOf D i k u k else 0) := by
    split_ifs with h
    · have hvj := h.1; have huk := h.2
      simpa [hvj] using (coeffOf_f D i k u k hi hk hkn hu huk hkn).symm
    · rfl
  -- ... similarly for IB, IIA, IIB ...
  rw [h_IA, h_IB, h_IIA, h_IIB]
```

## Key insight: each dite term is handled independently

The 4 `dite` terms in `half_deriv_cond` each have their own condition binder `h`.
By handling each one separately with `split_ifs`, the condition `h` is accessible
in the true branch, allowing `coeffOf_f` to be applied.

The `h.2` (inequality proof) from the dite binder provides the proof needed for
the MatIdx construction in `coeffOf_f`.

## Why calc + ring fails

```lean4
-- ❌ This approach fails:
rw [h_expand, hD, hId]
repeat (rw [h_if_distrib])
ring  -- ring can't handle dite binder terms
```

After `rw [hD, hId]`, the goal has dite terms from `hD` with binder `h` AND
ite terms on the RHS. `h_if_distrib` converts each RHS ite to dite form, but
the resulting expression has dite binders mixed with ring operations, which
`ring` (working on polynomial expressions) cannot normalize.

## Status

This pattern is used in `ToNCoeff.lean:coeffOf_cond` (✅ complete).
The same technique applies to `coeffOf_cond_zero` (🟡 frozen as Bracket.lean).
