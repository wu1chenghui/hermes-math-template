# Comprehensive Audit Layers for Lean Formalization Projects

> 2026-06-30: Taxonomy developed while auditing the `e/` Lean formalization.

## When to Use
When the user asks to "audit the project" or "verify the mathematics" beyond simple build checks.
Progress through layers in order. Stop when the user signals they want to move from
"verifying Lean" to "understanding the mathematics."

## Layer 0: Engineering
- Clean build (`lake clean && lake build`)
- Stale olean cache detection
- Module count, declaration count, line count
- Phantom API detection (#check every lemma before assuming it exists)

## Layer 1: Architecture
- Import DAG (module-level)
- Call graph (declaration-level)  
- Dead module verification (0 callers AND 0 importers)
- Module status classification (proof-critical / support / historical / dead)
- Tactic usage profile (omega, ring, simp, etc.)
- Mathlib API usage inventory
- Open namespace audit

## Layer 2: Proof Structure
- Theorem dependency DAG (from main theorem to leaves, tracing proof bodies)
- Import graph ≠ proof graph distinction
- Sorries-on-path analysis
- Hidden assumption audit (every logical step has a theorem)
- Converse audit (bidirectional links where needed)

## Layer 3: Mathematical Trust
- Model audit (Lean definition ↔ mathematical definition)
- Specification audit (theorem proves exactly what's needed)
- Freedom audit (all degrees of freedom classified)
- Coverage audit (every object classified exactly once)

## Layer 4: Mathematical Understanding
- Origin audit (mathematical source of each lemma)
- Derivation audit (why the proof has this shape)
- Discovery audit (obstacle → response chain)
- Invariant audit (mathematical invariants distinguishing parameters)
- Independent structural derivation (root system, Serre relations, CE cohomology)

## Layer 5: Proof Structure (Paper-Ready)
- Reverse reconstruction (derive from roots without Lean)
- Propagation lemma verification (replace case dispatch with induction)
- CE Propagation Theorem (simple roots ∈ I ⇒ all roots ∈ I)
- Boundary Classification Theorem (standalone mathematical statement)
- Parallel proof audit (paper proof ↔ Lean proof)

## Rules
1. Do NOT skip layers — the user values systematic progression
2. Distinguish "proven" from "structural explanation" — the user rejects overstated claims
3. Do NOT add more audits once Layers 0-3 are complete — the user wants to move toward mathematical understanding, not more verification
4. Prefer "proposition + one mathematical reason" format over exhaustive enumeration
5. The user prefers Chinese for discussion/strategy, English for Lean code/lemma names
