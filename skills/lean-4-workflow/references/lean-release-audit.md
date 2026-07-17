# Lean Release Audit Methodology

When mathematical proof is complete and `lake build` passes, perform a
structured release audit before declaring the project "done."  The goal is
to catch engineering-level issues that survive green builds.

## Priority Order

### 1. Confirm the final proof route is unique (highest priority)

- grep for all symbols in the final theorem's dependency chain
- Build a table: symbol → location → callers
- Identify VESTIGIAL routes: old designs that were never completed,
  alternative paths that were abandoned
- **Goal**: exactly ONE active path from definitions to the target theorem
- Do NOT trust AGENTS.md claims about "D4 complete" without verification

### 2. Dead-Code Audit

- Build a table: file → theorem → consumers (grep across entire project)
- Mark everything with 0 consumers as LEGACY (do NOT delete yet)
- Special attention to pre-existing sorries in imported-but-unused modules

### 3. Import DAG Verification

- Draw the module dependency graph for E.lean
- Check for: cycles, redundant imports, legacy imports, dual paths to same result
- Verify no module imports both an old and new version of the same concept

### 4. API Freeze

- Identify the Public API (paper-facing symbols)
- Identify Internal (implementation detail)
- Document the split clearly in AGENTS.md

### 5. Memory / AGENTS Sync

- Verify Memory entries match the actual source code, not cached claims
- When architecture changed mid-development, add an explicit "Architecture
  Revision" section in AGENTS.md documenting old vs new route
- Update Memory to prevent future agents from resurrecting dead routes

### 6. Build Verification

- Run `lake build` (full) — NOT `lake build <Module>` — and verify 0 errors
- Be aware: `lake build` with stale `.olean` cache can SILENTLY pass with
  missing typeclass instances or API-incompatible calls. If AGENTS.md claims
  "0 errors" but `lake build <Module>` fails, the claim was based on stale cache.
- **NEVER `lake clean`** unless you have 30+ minutes for full mathlib rebuild

## What NOT to do

- Do NOT delete dead code during the audit — mark it LEGACY, archive later
- Do NOT refactor files or rename namespaces
- Do NOT change the directory structure
- The build is a stable artifact; preserve it while documenting its state

## LEGACY Marking Convention

```lean
/-- LEGACY — kept for historical architecture documentation only.

    **This is NOT part of the final proof chain.**  The actual proof
    uses <alternative route>.  Current status: <placeholder/proved>.
    0 callers.  Superseded by <new lemma> in <location>. -/
```
