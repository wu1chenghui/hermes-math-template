# Release Audit Methodology

Systematic 5-priority post-proof audit for Lean formalization projects.
Perform AFTER `lake build` passes with 0 errors, BEFORE declaring completion.

## Priority 1: Confirm new proof route has completely replaced old route

When the final proof changed approach mid-project (e.g., from Equiv decomposition
to linear-map range analysis), verify the old route has zero consumers.

**Actions:**
1. grep all `.lean` files for every symbol in the OLD route
2. For each symbol: count callers, classify as LIVE (has consumers) or DEAD (0 callers)
3. Confirm the NEW route's DAG is a single connected path from inputs to theorem
4. Flag any symbol that appears in BOTH routes (potential dual-path ambiguity)

**Target:** exactly ONE proof path remains. No dual pipelines.

## Priority 2: Dead-code audit

Generate a table of all theorems with zero consumers. Do NOT delete immediately.

**Actions:**
1. For each `.lean` file, list every theorem/lemma
2. Search globally for each symbol name across all `.lean` files
3. If `consumers = 0` (excluding the declaration itself), mark as `LEGACY`
4. Produce a table: `| File | Theorem | Consumers | Status |`

**Rule:** do NOT delete during audit. Mark only. Archive in a unified pass later.

## Priority 3: Import DAG verification

Verify the project's import graph has no cycles, no legacy imports, and no dual paths.

**Actions:**
1. Read the root module (e.g., `E.lean`) for the full import list
2. Trace transitive imports for each submodule
3. Draw the DAG showing which pipeline each module belongs to
4. Check for:
   - Circular imports (A imports B imports A)
   - Redundant imports (module imported both directly and transitively)
   - Dead imports (imported module whose symbols are never used)

## Priority 4: API Freeze

After proof completion, identify which symbols form the stable public interface.

**Actions:**
1. List all exported symbols from the main theorem module
2. Classify each as:
   - **STABLE API**: the main theorem + key definitions that papers will reference
   - **IMPLEMENTATION DETAIL**: supporting lemmas, internal plumbing
3. The stable API should be minimal — typically 5-10 symbols

This prevents future maintenance from accidentally breaking the interface
that papers and downstream consumers depend on.

## Priority 5: Memory/AGENTS synchronization

After architecture changes, update persistent documentation so future sessions
don't try to revive dead routes.

**Actions:**
1. Check Memory entries for stale references to old routes
2. Update Memory with the actual completed route
3. Append an "Architecture Change" checkpoint to AGENTS.md documenting:
   - What the OLD plan was
   - What the NEW route actually is
   - Why the change happened
   - A clear directive: "Do NOT attempt to complete [old symbol]"

## Concrete Patterns

### LEGACY marker format

Use docstring annotations, not deletion. Format:

```lean
/-- LEGACY — kept for historical architecture documentation only.

    **This is NOT part of the final proof chain.** [reason].
    
    Current status: [placeholder / dead code]. 0 callers.
    Superseded by [replacement]. -/
theorem old_theorem ...
```

```lean
/-- LEGACY — 0 callers.  Kept for potential future use in [scenario].

    [what it proves and why it may be useful later]. -/
lemma old_lemma ...
```

## Pitfalls

### Pitfall: `lake clean` in mathlib projects = 30+ minute rebuild

`lake clean` deletes ALL `.olean` files including mathlib's ~3000 modules.
A full rebuild from scratch compiles every transitive dependency and takes
30+ minutes. **NEVER run `lake clean`** for diagnostic purposes. Prefer:
- `lake build E.Classification.Centering` — rebuilds only the target module
  and its dependencies if their oleans are invalidated
- Delete individual `.olean` files if you need to force recompilation of a
  specific module

### Pitfall: AGENTS.md build-success claims can be stale (olean cache)

When AGENTS.md says "lake build = 2967 jobs, 0 errors", this may reflect a
STALE `.olean` cache — `lake build` won't recompile modules whose `.olean`
timestamp is newer than the source. If Centering.lean was never recompiled
after its last code change (because the olean was preserved from an earlier
passing build), errors can lurk silently.

**Always verify with a targeted rebuild:** `lake build <SpecificModule>`
to force recompilation of the module you're auditing. Do not trust
project-wide `lake build` output alone for release-readiness claims.

### Pitfall: pre-existing errors masked by unchanged olean

When doing a release audit, the first targeted `lake build <Module>` may
reveal errors that were always present but masked by cached oleans. These
are NOT caused by your patches if you only changed comments/docstrings.
The build was never truly clean — the AGENTS.md claim was wrong.
