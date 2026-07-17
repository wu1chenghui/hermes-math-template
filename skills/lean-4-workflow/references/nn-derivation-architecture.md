# NnDerivation Architecture — Leibniz Axiom Pattern

Discovered during the June 2026 refactoring of the 1/2-derivation project.
This reference captures the architectural pattern for future Lean projects.

## The Problem

`HalfDerivation` (old) had:
- `coeff : MatIdx n → MatIdx n → F` as primitive DATA
- `half_deriv_cond` as the ONLY axiom (single-source split decomposition with factor 2)

This created a 204-line `sorry` for `bracket_zero_coeff` — the coefficient-level
bracket identity for commuting sources. The `half_deriv_cond` axiom controls
SPLITTING geometry (vertical decomposition) but cannot access COMMUTING geometry
(transverse coefficient propagation).

## The Solution

Replace `HalfDerivation` with `NnDerivation`:
- `coeff : ℕ → ℕ → ℕ → ℕ → F` as data (same representation, new semantics)
- `leibniz` as the ONLY axiom: D([x,y]) = [D(x), y] + [x, D(y)] at coefficient level

Key equation:
```lean4
leibniz (i j k l u v : ℕ) ... :
  bracketSource coeff i j k l u v = bracketIdentity coeff i j k l u v
```

Where:
- `bracketSource` = coefficient of E_uv in D([E_ij, E_kl]) — LHS of Leibniz
- `bracketIdentity` = coefficient of E_uv in [D(E_ij), E_kl] + [E_ij, D(E_kl)] — RHS

## Why It Works

When [E_ij, E_kl] = 0 (commuting sources, j≠k, i≠l):
- bracketSource = 0 (since [E_ij, E_kl] = 0 → D(0) = 0)
- Therefore bracketIdentity = 0 → immediate `bracket_zero_coeff`

This is a ONE-LINE proof:
```lean4
theorem bracket_zero (D : NnDerivation F n) ... (hj_ne_k : j ≠ k) (hi_ne_l : i ≠ l) :
    bracketIdentity D.coeff i j k l u v = 0 := by
  have h := D.leibniz i j k l u v ...
  unfold bracketSource at h
  simp [hj_ne_k, hi_ne_l] at h
  exact h.symm
```

## Degeneration Pattern

With the Leibniz axiom, the entire theorem hierarchy collapses:

| Layer | Old (HalfDerivation) | New (NnDerivation) |
|-------|---------------------|-------------------|
| `bracket_zero_coeff` | 204-line `sorry` | 1-line theorem |
| `coeffOf_cond` | Factor 2, complex if-conds | No factor 2, simple if-conds |
| `only_A/B/C/D` | Case analysis + compat | bracket_zero_coeff corollary |
| `achain/cchain/diag` | 6 theorems, factor 2 | coeffOf_cond corollaries |
| `bridge_left/right` | Length3-specific | Chain corollaries (1-line each) |
| `pair_cancel_*` | Cross-source machinery | bracket_zero_coeff corollaries |

## Architectural Principles

1. **Axiom = Mathematical Law**: Choose the axiom that matches the mathematical
   definition (Leibniz rule for derivations), not a computational convenience
   (split decomposition). The RIGHT axiom makes everything else a corollary.

2. **Projection before Coefficient**: Define the operator-level identity first,
   then project to coefficients. This makes coefficient-level theorems
   into `simp` expansions of the operator identity.

3. **No factor 2**: The factor 2 in `half_deriv_cond` was an artifact of the
   1/2-derivation normalization. Using full derivations (Leibniz) eliminates it
   and simplifies all downstream algebra.

4. **Strict unidirectional dependencies**: Each file imports only files below it:
```
MatrixBasis → BracketFormula → NnDerivation → Projection → Coefficient
                                                              ↓
                                              Only → Chain → Bridge → Pair
```

## When to Apply This Pattern

Consider refactoring to an operator-level axiom when:
- A lemma is stuck because the current axiom doesn't give enough information
- The missing information is about TWO sources interacting (not one source)
- The mathematical definition has a natural operator-level law (Leibniz, Jacobi, etc.)
- The coefficient-level version has unnecessary factors or complex conditions
