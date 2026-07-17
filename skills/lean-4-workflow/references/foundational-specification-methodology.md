# Foundational Specification Methodology

> Established 2026-06-30. This methodology emerged from a fundamental re-audit
> of the half-derivation classification project after v1.0-proof-complete.

## Core Insight

The dim=n error was not a missing case in a proof. It was a **failure to
generate the full constraint space SEq** — a class of half-Leibniz equations
(non-local commuting pairs) was omitted from the equation universe.

This led to a restructuring from "proof summaries + audit documents" to a
formal **Foundational Specification with numbered Phases**.

## The Phase Pipeline

```
Phase 0: Objects          — FOUNDATIONAL_SPECIFICATION.md
Phase A: Solution Space    — HalfDer basis
Phase B: Constraint Space  — SEq generators
Phase C: Morphology        — Generator decomposition
Phase D: Syzygy Framework  — Relations (Established + Conjectured)
Phase D′: Proof Package    — Prove all conjectures
Phase E: Basis Selection   — Candidate SEq basis
Phase F: Coordinate System — Evaluation matrix M
Phase G: Main Theorem      — Assembly
```

Architecture (0–F) → Proof (D′) → Assembly (G).

## Object Separation Principle

Every mathematical entity belongs to exactly ONE category:
- algebraic object
- linear map
- image
- kernel
- coordinate representation
- logical statement

Transformations between categories require explicit morphisms.
No cross-category identification without one.

## Phase Discipline

1. **Dependency Rule**: Each Phase depends only on Phases above it.
   No circular dependencies. 0→A→B→C→D→D′→E→F→G.

2. **Conjecture Dependency Rule**: Any reference to a Conjecture must be
   explicitly annotated: "Assuming Conjecture D.3.x". Conjectures may NOT
   be cited as established facts.

3. **Interface Contracts**: Each Phase declares exactly which prior Phase
   objects it may reference. Phase F, for example, may only reference
   B_cand (Phase E) and {e_k} (Phase A), not Ψ or T0–T3 families.

4. **Blueprint before document**: For any new Phase, first produce a
   one-page Blueprint answering foundational questions. Do not write the
   full document until the Blueprint is reviewed.

5. **Audit before freeze**: Every Phase is audited before freezing.
   Audit types:
   - Foundational Audit (Object/Arrow/Dependency/Uniqueness Closure)
   - Responsibility Audit (is each section doing only its own job?)
   - Theorem Audit (classify every claim: Definition/Observation/Conjecture/Theorem)

6. **Freeze with stamp**: Frozen documents carry a `**Status: FROZEN**`
   header. No modifications after freeze.

## Audit Taxonomy

### Foundational Audit (Phase 0.5)
- Object Closure: Every object traceable to Phase 0 definitions
- Arrow Closure: Every morphism has explicit domain/codomain/construction
- Dependency Closure: DAG of Phases, no reverse edges
- Uniqueness: No two names for the same object

### Responsibility Audit (Phases A–C)
Check that each section does only its own job:
- No premature decomposition (Phase B)
- No "force"/"therefore" conclusions (Phase C)
- No Independence/Rank/Minimality (Phases B–D)

### Theorem Audit (Phase D+)
Classify every claim:
- **Definition**: stipulative, no truth value
- **Observation**: immediately verifiable from definitions
- **Conjecture**: believed true, not yet proved
- **Theorem**: proved (or proof sketch provided)

## Document Structure Template

Each Phase document follows this structure:
```
# Phase X: Title
> Status: FROZEN | DRAFT
> Primary mathematical object: ...
> Interface Contracts: ...
Scope (establishes / does NOT establish)
§1–§N: Content
Exported Results (table with Status column)
NOT exported (table with Belongs to column)
```

## Key Terminology

- **Candidate**: Provisional, pending Phase D′ proof (e.g. "candidate basis",
  "candidate irreducible").
- **Established**: Proved within the current Phase.
- **Conjectured**: Stated as Conjecture D.x.x, proof deferred to D′.
- **Conditional**: Depends on unproven conjectures.
- **Structural**: Follows from the organization of indices/supports,
  not from algebraic computation.

## The dim=n Error: Precise Location

In the new framework, the error is precisely located:
```
SEq_{dim=n} = Im(π*∘Ψ♯|_{restricted pairs}) ⊊ Im(π*∘Ψ♯) = SEq
```
Missing: φ_{E₁, E_{n-1}, 1, n} (the F_eq functional from a non-local commuting
pair). The restricted domain excluded pairs with non-local index overlap.

This is a **generator omission**, not a proof error.
