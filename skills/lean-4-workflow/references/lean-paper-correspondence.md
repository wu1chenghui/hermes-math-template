# Lean–Paper Correspondence Methodology

> When a conjectured proof strategy fails but the Lean codebase implements a
> working mechanism, do NOT try to prove the conjecture. Establish correspondence.

## The Pattern

1. **Audit the Lean code.** Read the key lemma and trace it to the axiom it invokes.
2. **Compute a concrete expansion.** Pick a small parameter set, expand fully.
3. **State the correspondence.** E.g. `coeffOf_cond = Ψ(E_ik, E_kj, T) = 0`.
4. **Inherit Lean results.** Paper proves equivalence, then cites Lean.

## Three convergence paths

Every proof obligation resolves via one of:
- **Proved directly** (explicit computation).
- **Established via correspondence** (Lean lemma = paper equation).
- **Vacuous after structural analysis** (fewer non-zero objects than assumed).

## When to use this

Use when a conjectured formula resists proof and the Lean code has a different
mechanism. Do NOT keep trying to prove the conjectured formula — the Lean
mechanism is likely the correct one and the conjecture was wrong.

## Key Example

The conjectured `φ_i = 2·φ_{i+1}` (simple BL→BL propagation) was disproven by
explicit Ψ-expansion. Lean's `coeffOf_cond` (split-k recursion) was correct.
The identity `coeffOf_cond = Ψ(E_ik, E_kj, T) = 0` unified paper and Lean.
