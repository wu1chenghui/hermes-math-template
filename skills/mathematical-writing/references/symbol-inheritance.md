# Symbol Inheritance for Derivation-Classification Papers

Rule: match Ou-Wang-Yao (2007) first. If Ou doesn't have it, use
Kaygorodov-Khrypchenko (2023). Only keep our notation if neither has
a precedent.

## Inheritance Table

| Concept | Ou | KK | Our final | Source |
|---------|-----|-----|-----------|--------|
| Lie algebra | N(n,R) | T_n(F) | N(n,F) | Ou (field specialisation) |
| Matrix units | Eij | eij | E_{ij} | Ou |
| Half-derivation map | φ | ϕ | φ | Ou |
| Half-derivation space | — | Δ(L) | Δ(N(n,F)) | KK |
| Linear span | — | ⟨·⟩ | ⟨·⟩ | KK |
| Identity/scalar derivations | — | trivial 1/2-derivations | identity scaling | KK (pending) |
| Centered complement | — | — | centered / Δ₀ | New concept |
| Proof end marker | □ | □ | □ | Both |
| "i.e." — symbol subst | — | — | allowed | New rule (2026-07-11) |
| "i.e." — prose explanation | — | — | forbidden | Both |
| "whence" | — | yes | allowed | KK |
| "organized as follows" | yes | — | allowed (natural prose, not enumeration) | Ou |

## Decisions Made (2026-07-11)

1. **N(n,F)** over N_n: aligns with Ou's N(n,R). Field F replaces ring R.
   N(4,F) for the n=4 exceptional case.

2. **E_{ij}** over e_{ij}: follows Ou. The braces are a LaTeX necessity,
   visually equivalent to Ou's Eij.

3. **φ** over ϕ: follows Ou. Both papers use a single-stroke Greek phi.

4. **⟨·⟩** for linear span: follows KK. Ou uses no special notation for span.

5. **Δ(L)** for half-derivation space: follows KK. Ou doesn't study
   half-derivations.

6. **"centered"** is a new concept not in either reference. Kept because
   the decomposition φ = s·id + φ₀ with φ₀ centered has no precedent
   in the half-derivation literature.

7. **"trivial"** (KK's term for s·id) under consideration for future revision.
   Currently using "identity scaling."

## Notation Never Adopted

| Notation | Reason |
|----------|--------|
| N_n | Dropped in favour of N(n,F) — aligns with Ou |
| N_4 | → N(4,F) |
| F1/F2 | Internal code names — replaced by descriptive "Pair with E₁₂" / "Pair with E_{n-1,n}" |
| Regime | Non-standard — replaced by "If u=..." prose |
| β_i | Conflicts with KK's β_i — renamed ω_i |
| σ_k | Conflicts with Dynkin involution σ — renamed τ_k |
