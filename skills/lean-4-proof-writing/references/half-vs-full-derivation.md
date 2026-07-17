# Half-Derivation vs Full Derivation — Critical Distinction

**Discovered during 1/2-derivation classification refactoring (2026-06-18).**

## The Two Types

| | 1/2-Derivation (old `HalfDerivation`) | Full Derivation (new `NnDerivation`) |
|---|---|---|
| **Axiom** | `half_deriv_cond`: 2δ([x,y]) = [δ(x),y] + [x,δ(y)] | `leibniz`: D([x,y]) = [D(x),y] + [x,D(y)] |
| **Factor 2** | Present in axiom | None |
| **Identity map** | ✅ I(E_ij) = E_ij IS a 1/2-derivation | ❌ I(E_ij) = E_ij is NOT a full derivation |

## Verification

For δ = I (identity map), x = E_ik, y = E_kj:
- 1/2-derivation: 2·δ([x,y]) = 2·E_ij. [δ(x),y] + [x,δ(y)] = [E_ik,E_kj] + [E_ik,E_kj] = E_ij + E_ij = 2·E_ij. ✅
- Full derivation: D([x,y]) = E_ij. [D(x),y] + [x,D(y)] = 2·E_ij. 1 ≠ 2 for char≠2. ❌

## Impact on Classification

The classification theorem `dim Der_{1/2}(N_n) = n` is about **1/2-derivations**.
Our new `NnDerivation` models **full derivations**.

These are genuinely different objects for char ≠ 2. The relationship between them
on N_n needs to be clarified before porting:
- `all_adjacent_diag_equal` — different truth values in each setting
- `diag_scalar` — definition depends on which type we're using

## Recommendation

Before continuing Lean work on Classification:
1. Determine the correct bijection between 1/2-derivations and full derivations on N_n
2. Decide whether to adapt `NnDerivation` to model 1/2-derivations, or
3. Prove the classification for full derivations and relate to 1/2-derivations
