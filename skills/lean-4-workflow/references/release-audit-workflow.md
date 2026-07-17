# Release Audit Workflow for Lean Formalization Projects

When the main theorem is proven (all live sorries closed, `lake build` passes),
do this BEFORE claiming completion or starting cleanup.

## Priority 1: Confirm the proof path is unique

- Grep for ALL symbols in the claimed "final" proof chain
- Check if old/alternative proof paths still exist as dead code
- Generate a dependency table showing which symbols are LIVE vs LEGACY
- Goal: confirm there is exactly ONE active proof pipeline

## Priority 2: Dead-code audit (don't delete yet)

Generate a table:

| File | Theorem | Consumers | Status |
|------|---------|-----------|--------|

For each row where consumers = 0, mark as LEGACY.
Do NOT delete — archive later in a unified pass.

## Priority 3: Import DAG verification

- Run `lake env lean E.lean` (or targeted build)
- Map the full import DAG
- Confirm: no cycles, no dual paths, no legacy imports in the active chain

## Priority 4: API Freeze

Document Public vs Internal symbols.
Public: what the paper references.
Internal: implementation details not part of the mathematical claim.

## Priority 5: Documentation sync

- AGENTS.md: add Architecture Revision chapter
- AGENTS.md: add Final DAG visualization
- Memory: update to reflect actual (not planned) state

## What NOT to do

- ❌ Delete orphaned theorems immediately
- ❌ Refactor files
- ❌ Rename namespaces
- ❌ Restructure directories

The project is in a stable state. Premature cleanup risks breaking the build.
First complete the audit, THEN archive legacy code in a unified pass.

## Pitfall: AGENTS.md build-status claims may be unreliable

`lake build` reports success based on CACHED `.olean` files. A module that
never actually compiled with the current source can still show as "0 errors"
because no recompilation occurred. Before trusting any AGENTS.md checkpoint:

1. Touch a key `.lean` file to force recompile, or
2. Run `lake clean && lake build` (expensive: 30+ min for mathlib), or
3. Delete the specific `.olean` and rebuild that module alone

A `lake build` that completes in seconds with "0 errors" is a red flag.
