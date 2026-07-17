# Meta-Proof Object Audit (Phase 0)

> Established 2026-06-30. Precedes all other audits.
> Purpose: Define every mathematical object in the proof system before
> analyzing constraints, coverage, or rank. Prevents the dim=n class of
> errors — where five distinct object layers were compressed into one
> word ("equation"), causing an entire class of constraints to be missed.

---

## The dim=n Error, Re-Analyzed

**Surface symptom**: dim was computed as n instead of n+5.

**Actual root cause**: Object-level confusion. The constraint `[E₁, E_{n-1}] = 0`
produces a functional `φ_{E₁, E_{n-1}, 1, n}` in `HalfDer*` that relates F_a
and F_c. This functional was missing from the analysis because:

1. "Equation" was treated as a discrete object, not as a linear functional
   in a vector space
2. The mapping from pair → equation was assumed to be 1-to-1 (it's actually:
   one pair → one vector identity → many scalar identities → many functionals)
3. Commuting pairs ([x,y]=0) were assumed to give no information — but
   they produce non-trivial functionals when the RHS has index overlap

**The fix is not** "add F_eq to the list." The fix is: build the proof on
the correct mathematical objects from the start.

---

## The Five-Layer Object Pyramid

Every "constraint" in the half-Leibniz system is actually five distinct
objects. Conflating them is the root error.

```
Layer 0: PAIR           (E_ij, E_kl)                ∈ N_n × N_n
   ↓   Ψ
Layer 1: VECTOR IDENTITY  2T([x,y])−[Tx,y]−[x,Ty]   ∈ N_n
   ↓   π_uv (projection onto basis element E_uv)
Layer 2: SCALAR IDENTITY  π_uv(Ψ(x,y)(T)) = 0        ∈ F
   ↓   fix (x,y,u,v), vary T
Layer 3: FUNCTIONAL       φ_{x,y,u,v} : HalfDer → F   ∈ HalfDer*
   ↓   "= 0"
Layer 4: CONSTRAINT       φ(T) = 0                    ∈ {⊤,⊥}
```

- **Layer 0→1**: Algebra (bracket, linearity of T)
- **Layer 1→2**: Coordinate projection. One vector identity projects to
  d = n(n−1)/2 scalar identities (one per target E_uv)
- **Layer 2→3**: Currying: fix pair and target, get a linear functional on HalfDer
- **Layer 3→4**: Logic: assert the functional evaluates to zero for a given T

**Key fact**: What we call "an equation" is actually a functional (Layer 3).
One pair produces d functionals, not one equation.

---

## Complete Object Directory

### Base Spaces

| Object | Type | Description |
|--------|------|-------------|
| N_n | Vector space / Lie algebra | Strictly upper-triangular n×n matrices, dim = n(n−1)/2 |
| F | Field | char ≠ 2,3 |
| E_ij | Basis vector | 1 ≤ i < j ≤ n |
| [·,·] | Bracket | N_n × N_n → N_n, formula: [E_ij, E_kl] = δ_jk·E_il − δ_li·E_kj |

### Derivation Spaces

| Object | Type | Description |
|--------|------|-------------|
| Der(N_n) | Vector space | Full derivations δ: N_n → N_n |
| HalfDer | Vector space | 1/2-derivations T: condition 2T[x,y] = [Tx,y] + [x,Ty] |
| T(E_ij) | Vector in N_n | Image of basis element under T |
| coeff | HalfDer → (ℕ⁴ → F) | Raw coefficient (garbage at invalid indices) |
| coeffOf | HalfDer → (ℕ⁴ → F) | Normalized coefficient (0 at invalid indices) |

### The Equation Morphism Chain

| Object | Type | Description |
|--------|------|-------------|
| Ψ | N_n × N_n → (HalfDer → N_n) | Ψ(x,y)(T) = 2T[x,y] − [Tx,y] − [x,Ty] |
| π_uv | N_n → F | Coordinate projection onto E_uv |
| φ_{x,y,u,v} | HalfDer → F | φ = π_uv ∘ Ψ(x,y). A linear functional |
| EqSpace | Subspace of HalfDer* | span{ φ_{x,y,u,v} : all x,y,u,v }. The space of all theoretically possible constraint functionals |

**Critical**: `HalfDer = ⋂_{φ ∈ EqSpace} ker(φ)`. The definition of HalfDer
is the common zero-set of all functionals in EqSpace.

### Substructures

| Object | Type | Dim | Description |
|--------|------|-----|-------------|
| I | Subspace of N_n | 3 | span{E_{1,n-1}, E_{1,n}, E_{2,n}}. Max abelian ideal |
| diag(D) | Scalar | — | coeffOf(D,i,j,i,j) |
| centered | Subspace of HalfDer | n+4 | ∀i<j, diag(D,i,j) = 0 |
| kerΦ | Subspace of HalfDer | n+4 | Kernel of Φ operator |
| range(coeffOf_map) | Subspace | n+5 | Canonical image of HalfDer |
| Id scaling | 1-dim subspace | 1 | All diagonals equal |

### Constraint Analysis Objects (Later Phases)

| Object | Type | Phase |
|--------|------|-------|
| Unknown parameters | Basis of HalfDer/kerΦ | Phase 1 |
| Constraint functional | Element of EqSpace | Phase 2 |
| Equation syzygy | Linear dependence in EqSpace | Phase 4 |
| Minimal basis | Basis of EqSpace | Phase 5 |
| Incidence matrix | |Basis| × |Unknowns| | Phase 6 |

---

## The Meta-Proof Audit Phases

This is the correct order for establishing that a Lean formalization's
constraint system is complete and well-founded:

### Phase 0: Object Audit (THIS DOCUMENT)
Define every mathematical object and its type. Draw the morphism chain.
Confirm no object-level conflation.

### Phase 1: Unknown Universe
What space do the unknowns live in? What is a basis?
Answer: HalfDer's free parameters after all constraints are applied.
Expected answer: n+5 parameters (Id + A_i + B + C + D + E + F).

### Phase 2: Equation Universe
What is EqSpace? What is its dimension?
This is the space of ALL theoretically possible constraint functionals,
before any reduction.

### Phase 3: Equation Morphism
The map from Pair → Vector Identity → Scalar Identity → Functional.
Is it surjective onto EqSpace? What is its kernel?
Which pairs produce trivial (zero) functionals?

### Phase 4: Equation Syzygy
Linear dependencies among functionals in EqSpace.
Which equations are genuinely independent?
Which are consequences of others?
THIS is where dim=n failed — misclassifying dependence as independence.

### Phase 5: Minimal Basis
Prove existence of a basis for EqSpace.
Prove your chosen representatives form a basis (span + linear independence).
This is a THEOREM, not a construction.

### Phase 6: Coefficient Incidence
For each functional in the Minimal Basis, which Unknown parameters
does it involve? Build the incidence matrix.
Compute constraint degree per parameter.
Classify ZERO / DETERMINED / FREE as a partition derivable from the matrix.

---

## Rules for Meta-Proof Audits

1. **Object-first**: Define types before counting. Never say "there are N equations"
   without specifying which space they live in.

2. **No premature classification**: Phase 0 must not mention B, C, D, E, F.
   Those are Unknown Universe objects, not Equation Universe objects.

3. **Certificates, not summaries**: Every claim ("covered", "independent", "complete")
   must be derivable from the mathematical objects, not from narrative reasoning.

4. **Equation = functional**: An equation is not a pair of basis elements.
   It is a linear functional φ ∈ HalfDer*. Multiple pairs can produce the
   same functional.

5. **Commuting ≠ trivial**: [x,y]=0 does NOT mean the half-Leibniz equation
   is 0=0. The RHS [Tx,y] + [x,Ty] may be non-zero. The F_eq constraint
   comes from a commuting pair.

6. **EqSpace is a vector space**: Equations are not a set — they span a
   subspace of HalfDer*. Linear dependence (syzygy) is a structural property
   of this subspace, not an afterthought.

7. **Minimal basis is a theorem**: You must PROVE your chosen set of equations
   spans EqSpace and is linearly independent. You cannot assert it by
   construction.
