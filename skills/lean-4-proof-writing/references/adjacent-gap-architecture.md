# Adjacent-Source Gap Architecture

## Two Independent Mechanisms

The project's `half_deriv_cond` and `coeffOf_cond_zero` are NOT derivable from each other:

| Mechanism | Controls | Encoded in |
|-----------|----------|------------|
| Splitting geometry | Vertical decomposition of source via bracket | `half_deriv_cond` / `coeffOf_cond` |
| Commuting geometry | Transverse coefficient flow across disjoint sources | `coeffOf_cond_zero` |

## Why Jacobi Cannot Substitute

Jacobi identity preserves the left source index along bracket chains. For adjacent source `E(i,i+1)`, all Jacobi-derived expressions have first index `i`. Targets with first index `u ≠ i` (transverse propagation) are **information-theoretically unreachable** from splitting axioms alone.

This is a structural obstruction, not a missing proof technique.

## `split_ifs` with `rename_i` Pitfall

When using `split_ifs` with the `with` syntax on multiple conditions:
```lean4
split_ifs with hA hB hC hD
```
The false branches produce `¬(condition)` = `condition → False` (function type). Do NOT `rcases` these. `rename_i` renames ALL hypotheses in order, making it easy to confuse true/false branch hypotheses.

**Prefer `by_cases` for each condition individually** — cleaner and avoids this issue.

## Embedding Technique (typeA/typeB)

For adjacent source `E(i,i+1)`, vanishing at targets sharing an index:
- **typeA**: target second index = `i+1`. Embed in non-adjacent source `E(i,i+2)` via `coeffOf_cond` with `k=i+1`.
- **typeB**: target first index = `i`. Embed in non-adjacent source `E(i-1,i+1)` via `coeffOf_cond` with `k=i`.

These are the ONLY adjacent-source coefficient identities provable from `coeffOf_cond` + `coeffOf_nonadjacent` alone. All other cases require `coeffOf_cond_zero`.
