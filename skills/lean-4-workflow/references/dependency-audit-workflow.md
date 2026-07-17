# Dependency Audit Workflow

When the sorry count drops to ~15 or fewer, pause and do a full dependency
audit before attacking any single sorry.  This prevents wasted effort on
sorries blocked by unresolved upstream nodes.

## The recipe

1. **List all sorries** with enclosing lemma name and a 1-line description.
   Search: `search_files(pattern="sorry", target="content", file_glob="*.lean")`
   on the frontier file.

2. **Classify into three tiers**:
   - **Core SCC**: sorries that require new proofs and may form mutual
     dependencies.  Map which calls which.
   - **Assembly/wiring**: sorries in dispatch or skeleton code that will be
     trivially filled once core lemmas are proved.  Mark with the name of the
     core lemma they await.
   - **Deferred**: sorries that legitimately belong to a later proof section
     (§D centering, all_diag_equal).  Do not touch these.

3. **Draw the dependency graph** for core SCCs only.  Identify:
   - Independent clusters (no cross-dependencies)
   - Single-node lemmas (can be proved in isolation)
   - Mutual-dependency SCCs (must be proved jointly)

4. **Verify 3-for-1 and N-for-1 opportunities**.  When multiple sorries are
   instances of the same parameterized lemma (e.g., `boundary_family_right`
   closing two T3 sorries), prioritize that lemma — it's the highest-leverage
   single proof.

5. **Order by leverage and independence**.  Pick the highest-leverage
   independent node first.

## Example (from 2026-06-27 session)

Starting state: 0 errors, 14 sorries in ImageContainment.lean.

```
Core SCC 1: boundary_family_right (1 sorry → closes 2 T3 sorries)  ← 3-for-1
Core SCC 2: width_c k=2 (2 sorries)
Core SCC 3: internal_det_coupling u≠2 (1 sorry)
Deferred:   4 sorries (§D centering / all_diag_equal)
Assembly:   4 sorries (dispatch + skeleton, waiting for core)
```

Result: Attack SCC 1 first (highest leverage).  The u=3 branch closed →
verified the 3-for-1 prediction (T3(3,n) eliminated).  u≥4 deferred.

## Anti-patterns

- **Don't attack assembly sorries first.**  They depend on core lemmas.
  Filling them early means either leaving placeholders or writing code that
  changes when core lemmas change signatures.

- **Don't attack deferred sorries.**  They belong to a different proof layer.
  Forcing their closure in the current layer creates structural cycles
  (e.g., proving all_diag_equal inside imageContainment).
