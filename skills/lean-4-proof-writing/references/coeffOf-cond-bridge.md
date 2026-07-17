# coeffOf_cond: dite→ite bridge for HalfDerivation → NCoeff

## Pattern

The `coeffOf_cond` lemma translates `HalfDerivation.half_deriv_cond` (which uses
MatIdx with dependent dite binders) to `NCoeff.cond` (which uses plain ℕ indices
with regular ite).  The proof bridges 4 dite terms to 4 ite terms, one at a time.

## Structure

```lean4
lemma coeffOf_cond (D : HalfDerivation F n) (i j u v : ℕ) ... (k : ℕ) (hk : i < k) (hk' : k < j) :
    (2 : F) * coeffOf D i j u v =
      (if v = j ∧ u < k then coeffOf D i k u k else 0)
    - (if u = k ∧ j < v then coeffOf D i k j v else 0)
    - (if v = k ∧ u < i then coeffOf D k j u i else 0)
    + (if u = i ∧ k < v then coeffOf D k j k v else 0) := by
```

## Key ingredients

1. **`coeffOf_f` lemma**: `coeffOf D i j u v = D.coeff ⟨i,j,...⟩ ⟨u,v,...⟩` when indices are valid.
   This collapses the ℕ-indexed representation to MatIdx representation.

2. **`half_deriv_cond`**: gives the identity with MatIdx terms and dite binders.

3. **`let` binders for proof terms** that appear repeatedly:

```lean4
  let x : MatIdx n := ⟨i, j, hi, hij, hjn⟩
  let y : MatIdx n := ⟨u, v, hu, huv, hvn⟩
  let hkn : k ≤ n := Nat.le_trans (Nat.le_of_lt hk') hjn
  let hin : i ≤ n := Nat.le_trans (Nat.le_of_lt hij) hjn
  let hpos_kj : 1 ≤ k := le_trans hi (Nat.le_of_lt hk)
```

4. **Expand LHS via `coeffOf_f` and `half_deriv_cond`**:

```lean4
  have h_left : coeffOf D i j u v = D.coeff x y := by
    dsimp [x, y]
    rw [coeffOf_f D i j u v hi hij hjn hu huv hvn]
  rw [h_left, h_cond]
  dsimp [x, y, MatIdx.left, MatIdx.right]
```

5. **Handle each of the 4 dite terms individually** with `split_ifs`, extracting
   condition components via `h.1`/`h.2` (NOT `rcases` — avoids binder issues):

```lean4
  have h_IA : (if h : v = j ∧ u < k then
      D.coeff ⟨i, k, hi, hk, hkn⟩ ⟨u, k, hu, h.2, hkn⟩ else 0)
    = (if v = j ∧ u < k then coeffOf D i k u k else 0) := by
    split_ifs with h
    · have hvj := h.1; have huk := h.2
      simpa [hvj] using (coeffOf_f D i k u k hi hk hkn hu huk hkn).symm
    · rfl
```

6. **Combine**: `rw [h_IA, h_IB, h_IIA, h_IIB]`

## Critical rules

- Use `h.1`/`h.2` NOT `rcases h with ⟨...⟩` — `rcases` destroys the binder and
  can cause `Unknown identifier` errors for variables that were part of the condition.
- Use `simpa [hvj]` with `.symm` because `coeffOf_f` gives `coeffOf = D.coeff` but
  the goal has `D.coeff = coeffOf`.
- Use `dsimp [x, y]` BEFORE `rw [coeffOf_f ...]` — the `let` binder needs to be
  expanded for the types to match.
- All 4 terms follow the same template: `split_ifs` → `have` components → `simpa` with `.symm`.
- The proof terms (`hkn`, `hin`, `hpos_kj`) must match the exact expressions that
  appear in the dite from `half_deriv_cond` — define them as `let` binders set to
  those exact expressions (e.g., `Nat.le_trans (Nat.le_of_lt hk') hjn`).

## The 4 dite→ite bridges

| Term | Condition | `coeffOf` call | Math meaning |
|------|-----------|---------------|--------------|
| IA | `v=j ∧ u<k` | `coeffOf D i k u k` | D(E(i,k)) at E(u,k) |
| IB | `u=k ∧ j<v` | `coeffOf D i k j v` | D(E(i,k)) at E(j,v) |
| IIA | `v=k ∧ u<i` | `coeffOf D k j u i` | D(E(k,j)) at E(u,i) |
| IIB | `u=i ∧ k<v` | `coeffOf D k j k v` | D(E(k,j)) at E(k,v) |
