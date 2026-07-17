# Sorrie 3 — 2-Equation Syzygy for u>i, v=n

Discovered 2026-06-28.  Closes the `adjacent_noncentral_offdiag_zero` case where
source=(i,i+1), target=(u,n), with **u > i** (target row BELOW source row).

## The Pattern

Two half-Leibniz equations with different sources but the SAME partner (i,i+1):

| Equation | Source | Partner | Target | Result |
|----------|--------|---------|--------|--------|
| Eq A     | (i, u) | (i,i+1) | (i, n) | coeff(i,i+1; u,n) = coeff(i,u; i+1,n) |
| Eq B     | (i+1,u) | (i,i+1) | (i+1,n) | coeff(i,i+1; u,n) = −2·coeff(i,u; i+1,n) |

Combined: `coeff = −2·coeff` → **3·coeff = 0** → in char≠3: **coeff = 0**.

## T3 Mechanism

`coeff(i,i+1; u,n)` NEVER appears in its own source's bracketIdentity (T1-T4) or bracketSource.
It appears ONLY as **T3** when the SOURCE has column = u and partner = (i,i+1):

```
T3 fires when: target_row = source_row AND source_col < target_col
             → coeff(partner; source_col, target_col)
             = coeff(i,i+1; u, n)   when source=(p, u), partner=(i,i+1), target=(p, n)
```

This is why the two equations use sources (i,u) and (i+1,u) — they have column u,
making T3 fire with partner (i,i+1) and target column n.

## Characteristic Dependency

- **char ≠ 3**: coefficient forced to zero
- **char = 3**: consistent with the char-3 exception in the main theorem

This is DIFFERENT from BoundaryRigidity which depends on char≠2.

## Comparison with BoundaryRigidity (u=i case)

|               | BoundaryRigidity | Sorrie 3 |
|---------------|-----------------|----------|
| Equations     | 3               | 2        |
| Targets       | (1,n), (2,n) ∈ I | (i,n), (i+1,n) ∉ I |
| Char dependence | char ≠ 2       | char ≠ 3 |
| Partner       | 3 different     | Same (i,i+1) |
| Variables     | a,b,g           | a,g      |

## Verified Lean Code (concrete: n=8, i=3, u=6)

```lean
-- Eq A: source=(3,6), ptr=(3,4), tgt=(3,8)
-- bracketSource = 0 (j=6≠c=3, i=3≠d=4)
-- bracketIdentity T3: u=3=3 ∧ 6<8 → coeff(3,4;6,8)

-- Eq B: source=(4,6), ptr=(3,4), tgt=(4,8)  
-- bracketSource = -coeff(3,6;4,8) (i=d: 4=4)
-- bracketIdentity T3: u=4=4 ∧ 6<8 → coeff(3,4;6,8)

-- Combined: coeff(3,4;6,8) = coeff(3,6;4,8) = -2*coeff(3,6;4,8)
-- → 3*coeff = 0 → (char≠3) coeff = 0
```

Full verification at the end of this file.

## Parameterization to General (i, u, n)

Requirements:
- `1 ≤ i < i+1 ≤ n` (adjacent source)
- `i < u < n` (target row between source row and n)
- Source (i,u) requires i<u ✓; Source (i+1,u) requires i+1<u ✓ (both satisfied since i<u)
- bracketSource simplifications: rely on standard index inequalities

The proof generalizes directly — all T1/T2/T4 are dead, T3 fires cleanly,
and the algebra (Eq A + Eq B → 3X=0) is parameter-free.

## Relation to Gap Map

In `adjacent_noncentral_offdiag_zero`, the u>i branch (line 140 sorrie) is
EXACTLY this pattern.  Combined with:

- u=i: BoundaryRigidity (3-equation, char≠2)
- u<i: symmetric pattern (TBD, expected to use similar T3 mechanism)
- i=n-1 (Pattern D): special 1-equation case

the v=n adjacent-source gap reduces to 2 remaining patterns (u<i and i=n-1).

---

## Full Lean Verification (concrete instance)

```lean
import E.Core.MatrixBasis
import E.Core.BracketFormula
import E.Core.HalfDerivation
import E.Core.HalfProjection
import E.Core.PhiOperator

open MatIdx PhiOperator

variable (F : Type) [Field F] [CharNeTwo F]

example (coeff : ℕ → ℕ → ℕ → ℕ → F) (hPhi : ∀ i j c d u v, Phi coeff i j c d u v = 0) 
    (hchar3 : (3 : F) ≠ 0) : coeff 3 4 6 8 = 0 := by
  have hPA := hPhi 3 6 3 4 3 8
  have hPB := hPhi 4 6 3 4 4 8
  unfold Phi at hPA hPB
  -- bracketSource A = 0
  have hbsA : bracketSource coeff 3 6 3 4 3 8 = 0 := by
    unfold bracketSource
    have h63 : (6 : ℕ) ≠ 3 := by omega
    have h34 : (3 : ℕ) ≠ 4 := by omega
    simp [h63, h34]
  rw [hbsA] at hPA
  -- bracketIdentity A: T3 fires, giving coeff(3,4;6,8); T2 fires, giving -coeff(3,6;4,8)
  have hbiA : bracketIdentity coeff 3 6 3 4 3 8 = (-coeff 3 6 4 8) + coeff 3 4 6 8 := by
    rw [bracketIdentity_eq_expanded]
    have h4 : (4 : ℕ) < 8 := by omega
    have h5 : (6 : ℕ) < 8 := by omega
    have h2 : (8 : ℕ) ≠ 4 := by omega
    have h6 : (8 : ℕ) ≠ 6 := by omega
    simp [h4, h5, h2, h6]
  rw [hbiA] at hPA
  -- hPA: 2*0 - ((-a) + b) = 0 → b = a
  have h_eqA : coeff 3 4 6 8 = coeff 3 6 4 8 := by
    apply sub_eq_zero.mp
    calc
      coeff 3 4 6 8 - coeff 3 6 4 8 = (-coeff 3 6 4 8) + coeff 3 4 6 8 := by ring
      _ = -(2*0 - ((-coeff 3 6 4 8) + coeff 3 4 6 8)) := by ring
      _ = -(0 : F) := by rw [hPA]
      _ = 0 := by ring
  -- bracketSource B = -coeff(3,6;4,8)
  have hbsB : bracketSource coeff 4 6 3 4 4 8 = -coeff 3 6 4 8 := by
    unfold bracketSource
    have h63 : (6 : ℕ) ≠ 3 := by omega
    simp [h63]
  rw [hbsB] at hPB
  -- bracketIdentity B: only T3 fires, giving coeff(3,4;6,8)
  have hbiB : bracketIdentity coeff 4 6 3 4 4 8 = coeff 3 4 6 8 := by
    rw [bracketIdentity_eq_expanded]
    have h1 : ¬ ((4 : ℕ) < 3) := by omega
    have h2 : (8 : ℕ) ≠ 4 := by omega
    have h3 : ¬ ((4 : ℕ) = 3) := by omega
    have h4 : (4 : ℕ) < 8 := by omega
    have h5 : (6 : ℕ) < 8 := by omega
    have h7 : (8 : ℕ) ≠ 6 := by omega
    simp [h1, h2, h3, h4, h5, h7]
  rw [hbiB] at hPB
  -- hPB: 2*(-a) - b = 0
  rw [← h_eqA] at hPB
  -- hPB: 2*(-b) - b = 0 → 3*b = 0
  have h3zero : (3 : F) * coeff 3 4 6 8 = 0 := by
    calc
      (3 : F) * coeff 3 4 6 8 = -((2 : F)*(-coeff 3 4 6 8) - coeff 3 4 6 8) := by ring
      _ = -(0 : F) := by rw [hPB]
      _ = 0 := by ring
  rcases eq_zero_or_eq_zero_of_mul_eq_zero h3zero with (h | h)
  · exact (hchar3 h).elim
  · exact h
```
