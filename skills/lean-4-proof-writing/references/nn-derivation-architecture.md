# NnDerivation Architecture — Leibniz Axiom Pattern

*Refactored 2026-06-18. Supersedes the old HalfDerivation-based architecture.*

## The Problem

The old `HalfDerivation` had `coeff` as primitive data and `half_deriv_cond` as the ONLY axiom:
```
(2)·coeff(i,j;u,v) = A − B − D + C    (split k decomposition)
```
This could NOT prove `bracket_zero_coeff` (the coefficient identity for commuting sources)
because `half_deriv_cond` only decomposes a SINGLE source.  The bracket identity
`[D(E),F] + [E,D(F)] = 0` for commuting E,F involved TWO sources and was irreducible.

## The Solution

Replace `HalfDerivation` with `NnDerivation`.  Same data (`coeff`), different axiom:

```lean4
structure NnDerivation (F : Type) [Field F] [CharNeTwo F] (n : ℕ) where
  coeff : ℕ → ℕ → ℕ → ℕ → F
  leibniz (i j k l u v : ℕ) (...) :
    bracketSource coeff i j k l u v = bracketIdentity coeff i j k l u v
```

The Leibniz rule `D([x,y]) = [D(x),y] + [x,D(y)]` projected to basis elements gives:
- **LHS** (`bracketSource`): `(if j=k then coeff(i,l;u,v) else 0) − (if i=l then coeff(k,j;u,v) else 0)`
- **RHS** (`bracketIdentity`): `bracketLeft + bracketRight` (the four-term A−B+C−D expansion)

## Key Consequences

### 1. `bracket_zero_coeff` — one line
When `[E_ij, E_kl] = 0` (j≠k and i≠l), `bracketSource = 0`, so `bracketIdentity = 0`:
```lean4
theorem bracket_zero_coeff (D : NnDerivation F n) ... :
    bracketIdentity D.coeff i j c d u v = 0 :=
  bracket_zero D i j c d u v ...
```

### 2. `coeffOf_cond` — no factor 2, simpler if-conditions
```
coeff(i,j;u,v) = (if u<k ∧ v=j then coeff(i,k;u,k) else 0)
               − (if u=k ∧ j<v then coeff(i,k;j,v) else 0)
               + (if u=i ∧ k<v then coeff(k,j;k,v) else 0)
               − (if u<i ∧ v=k then coeff(k,j;u,i) else 0)
```
Compare with old `half_deriv_cond` which had `(2)·coeff = ...` and extra constraints `u<k`, `j<v` etc. in every if-condition.

## File Structure (E/Core/)

| Layer | File | Purpose |
|-------|------|---------|
| 1 | `MatrixBasis.lean` | `MatIdx`, `bracket`, `CharNeTwo` |
| 2 | `BracketFormula.lean` | `bracketLeft/Right/Identity/Source` (coefficient-level, no NnDerivation dep) |
| 3 | `Derivation.lean` | `NnDerivation` with `leibniz` axiom; `bracket_zero` theorem |
| 4 | `Projection.lean` | `coeffOf` wrapper with bounds checking |
| 5 | `Coefficient.lean` | `coeffOf_cond`, `bracket_zero_coeff` (both theorems, not axioms) |

## Dependency Direction (strictly acyclic)

```
MatrixBasis → BracketFormula → NnDerivation → Projection → Coefficient
```

## Proof Pattern: coefficient conversion

When proving coefficient theorems that involve `coeffOf`, the standard pattern is:
1. Apply Leibniz to get an identity on `D.coeff` (raw, no bounds checking)
2. Expand `bracketIdentity`/`bracketLeft`/`bracketRight`
3. Convert each `D.coeff` term to `coeffOf` using `split_ifs` + `coeffOf_f`:
```lean4
have hA : (if cond then D.coeff i k u k else 0) =
         (if cond then coeffOf D i k u k else 0) := by
  split_ifs with hcond
  · rcases hcond with ⟨huk, hvj⟩
    have hcoeff : coeffOf D i k u k = D.coeff i k u k :=
      coeffOf_f D i k u k hi hik hkn hu huk hkn
    simp [hcoeff]
  · rfl
```
4. Close remaining algebraic identities with `ring`.

## Pitfall: `simp` on `bracketSource`

When applying Leibniz to a split `(i,k)` and `(k,j)`, `bracketSource` simplifies to
`D.coeff i j u v − (if i=j then D.coeff k k u v else 0)`.  `simp` alone cannot reduce
`i=j` to false; provide `hi_ne_j : i ≠ j` (from `hij : i < j` via `omega`).

## Relationship to Legacy

The legacy `HalfDerivation` type and all old files still exist in `E/HalfDerivation/`
and `E/` root.  They are preserved as reference but not imported by the new `E.lean`.
Backup: `/workspace/backups/E_backup_20260618_013535_refactor/`.
