# Nonadjacent Child Adjacent Pattern

## Problem

When proving coefficient vanishing for a nonadjacent source `(i,j)` with `j-i ≥ 2`,
the standard approach is width induction via `coeffOf_source_exists` + `induction'`.
But the induction setup (`revert`, `generalizing`, `induction'` from mathlib) is 
fragile and parameter-heavy — especially when the target column is `n` (v=n case).

## Key insight

`coeffOf_source_exists` splits the source at `k = i+1`. The 4 children are:

| Case | Child source | Child target | Source width | Resolution |
|------|-------------|-------------|-------------|------------|
| IA | `(i, i+1)` | `(u, i+1)` | **1 (adjacent!)** | `offdiag_zero_v_lt_n` (child v = i+1 < n) |
| IB | `(i, i+1)` | `(j, n)` | **1 (adjacent!)** | `adjacent_above_zero` (u=j > i+1) |
| IIA | `(i+1, j)` | `(i+1, n)` | `j-i-1` (nonadjacent) | recursion |
| IIB | `(i+1, j)` | `(u, i)` | `j-i-1` (nonadjacent) | `offdiag_zero_v_lt_n` (child v = i < n) |

**3 of 4 children are immediately resolved by existing lemmas with NO induction needed.**
Only IIA needs recursion.

## Why this pattern matters

- IA and IIB children have `v < n`, so `offdiag_zero_v_lt_n` applies directly.
- IB child has ADJACENT source `(i, i+1)`, so adjacent helper lemmas apply.
- This eliminates the need for a complex `induction'` with `generalizing i j u` for all but the IIA case.
- For IIA, simple source-width recursion suffices (each step reduces source width by 1).

## Implementation template

```lean
lemma nonadjacent_target_n_zero (D) (i j u) ... : coeffOf D i j u n = 0 := by
  by_contra! h_nonzero
  set k := i+1 with hk_ip1
  have h_src := coeffOf_source_exists D i j u n ... h_nonzero k ...
  rcases h_src with ⟨hnj, huk, hf⟩ | ⟨huk_eq, hjn_lt, hf⟩ | ⟨hui_eq, hkn_lt, hf⟩ | ⟨hui, hnk, hf⟩
  · -- IA: child (i,k; u,k), source width 1 (adjacent), target v=k<n
    -- → offdiag_zero_v_lt_n
    have h_child : coeffOf D i k u k = 0 := offdiag_zero_v_lt_n D ... 
    exact hf h_child
  · -- IB: child (i,k; j,n), source width 1 (adjacent), target (j,n) with j>i+1
    -- → adjacent_above_zero (u=j case for adjacent source)
    have hzero_adj : D.coeff i (i+1) j n = 0 := adjacent_above_zero D.coeff i j ...
    have hzero_coeffOf : coeffOf D i (i+1) j n = 0 := by rw [coeffOf_f ..., hzero_adj]
    apply hf; rw [hk_ip1, hzero_coeffOf]
  · -- IIA: u=i, child (k,j; k,n), source width ≥1, needs recursion
    ...
  · -- IIB: child (k,j; u,i), source width ≥1, target v=i<n
    -- → offdiag_zero_v_lt_n
    have h_child : coeffOf D k j u i = 0 := offdiag_zero_v_lt_n D ...
    exact hf h_child
```

## IIA sub-case analysis

The IIA child `(i+1,j)` → `(i+1,n)` has source width `j-i-1` and requires special handling:

| Parent condition | Child source width | Resolution |
|-----------------|-------------------|------------|
| `j-i ≥ 3` | `j-i-1 ≥ 2` (nonadjacent) | **recursion**: call `nonadjacent_target_n_zero` itself, measure `(j-i)-1 < j-i` |
| `j-i = 2` + `n ≥ i+3` | 1 (adjacent) | **BoundaryRigidity**: child has `u = source_row` at target col `n` |
| `j-i = 2` + `n = i+2` | 1 (adjacent) | **diagonal**: child target `(i+1,i+2)` = source → needs centered hypothesis |
| `i = 1` | any | **requires centeredness** (child target `(1,n) ∈ I`) |

The first two cases are Phi-only and clean. The last two require information that only the centered caller (`centered_image_in_I`) has. **Design rule**: keep `nonadjacent_target_n_zero` as a Phi-only lemma. Handle the `n=i+2` diagonal and `i=1` edge cases at the caller level, where `centered_diag_zero` is available.

## Design principle: Phi-only lemma + centered caller

```
nonadjacent_target_n_zero     ← Phi-only (no centered, no I_filtered, no kerΦ)
        ↑
        │  called by
centered_image_in_I           ← has centered D', dispatches n=i+2 / i=1 edges
```

This keeps the lemma portable and the proof clean. The lemma never needs to know about centeredness — the caller handles the boundary cases that require it.

## Related pitfalls

- **Don't use `induction'` + `generalizing`** for this pattern unless IIA is the only case.
  The `generalizing i j u` parameter ordering is fragile and hard to debug.
- **`subst hvn_eq` destroys section variable `n`** — use `rw [hvn_eq]` instead
  when `n` is a `variable` parameter. See SKILL.md pitfall section.
- **IA `h_notI` needs explicit proof**: `(u,k) ∉ I` doesn't follow trivially from
  `(u,n) ∉ I` when `k=n-1` and `u=1`. Use case analysis on `I_target_set` (but
  avoid `simp [I_target_set, hn3]` with `open MatIdx` — see SKILL.md MatIdx pitfall).
