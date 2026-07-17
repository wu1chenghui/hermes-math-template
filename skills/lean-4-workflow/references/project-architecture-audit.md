# Project Architecture Audit — Methodology

> 2026-06-30: Lessons from auditing the `e/` Lean formalization project.

## The Audit Hierarchy

When auditing a large Lean formalization project, progress through these layers:

### Layer 0: Clean Build
- `lake clean && lake build` — the ONLY trustable verification
- Stale olean caches mask errors — old proofs used phantom lemmas that don't exist
- Before trusting any "0 errors" claim: check olean timestamps, diff against backups

### Layer 1: Architecture
- Import DAG: which modules import which
- Call graph: which declarations are actually called by other modules
- Dead modules: 0 callers AND 0 importers — verify BOTH before deleting
- Module status: per-file classification (proof-critical / support / historical / dead)

### Layer 2: Proof Structure
- Theorem dependency DAG: from main theorem to leaves, through proof bodies
- IMPORT GRAPH ≠ PROOF GRAPH: modules imported but theorems never called
- Sorries-on-path: which sorries are on the actual proof graph
- Hidden assumptions: logical steps not backed by explicit theorems

### Layer 3: Mathematical Trust
- Model audit: does each Lean definition match the mathematical definition?
- Specification audit: does each theorem prove exactly what's needed?
- Freedom audit: are all degrees of freedom classified?
- Coverage audit: is every mathematical object classified exactly once?

### Layer 4: Mathematical Understanding
- Origin audit: what mathematical structure does each lemma reflect?
- Derivation audit: why does the proof have this shape?
- Discovery audit: how would someone discover this proof from scratch?
- Invariant audit: what mathematical invariants distinguish the parameters?

## Key Pitfalls Discovered

### "Max Type" is often misleading
The error "failed to synthesize instance of type class Max Type" can be:
1. A genuine universe issue (rare)
2. A cascade from a `LinearEquiv.finiteDimensional` direction error (more common)  
3. A `have`-type elaboration failure where the same expression works as a goal

Always verify which before concluding "Max Type = universe problem."

### `LinearEquiv.finiteDimensional` direction
`f.finiteDimensional` proves FD(codomain) from FD(domain). Using `.symm` reverses this.
Wrong direction causes cascading "Max Type" errors downstream.

### Dead module verification
A module with 0 callers is NOT necessarily safe to delete. Check IMPORTERS:
`search_files(pattern='import.*ModuleName', target='content')` across entire project.
In one audit, 11/24 "dead" candidates were imported by live code.

### `Module.finrank` on Submodule sup in `have` types
`Module.finrank F (sId ⊔ kerPhi ...)` as a `have` hypothesis TYPE triggers
elaboration failure. Same expression as goal (after `rw`) works fine.
Fix: eliminate `have` + `simpa`; inline proof on goal.
