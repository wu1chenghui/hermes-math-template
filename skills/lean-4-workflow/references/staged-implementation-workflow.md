# D3-style Staged Implementation Workflow

When the mathematical architecture is frozen but Lean proof engineering remains:

## The Pattern

1. **Step A — Lightweight skeleton revision** (BEFORE implementation)
   - Update only COMMENTS and docstrings to reflect the agreed architecture
   - Do NOT change Lean signatures, dispatch structure, or induction measure
   - Typecheck once, freeze if clean
   - Goal: align the text with the architecture, eliminate stale descriptions

2. **B1 — Freeze the induction skeleton** (API + framework)
   - Write the induction framework with explicit sorries for the body
   - The induction measure, dispatch, and API shape are FINAL after this step
   - Compiles with 0 error (sorries only in proof bodies)
   - Goal: lock the interface so B2/B3 can fill independently

3. **B2 — Recursive propagation** (easy edges first)
   - Fill all branches that close via simple mechanisms (char≠2, all-zero children)
   - Do NOT attempt terminal/cyclic-cluster cases
   - Goal: verify the induction framework is rich enough to propagate ALL recursive edges

4. **B3 — Terminal closure** (the hard cluster entries)
   - Fill the remaining cases that require cluster-specific lemmas
   - Each terminal case is a thin wrapper: split → child → cluster lemma
   - Goal: close the last exits; the theorem body is now sorry-free

## Why this works

- Each step compiles independently → errors are localized to the current step
- B2 verifies the induction is strong enough BEFORE investing in B3
- B3 failures can't break B2 or B1 → blame is narrow
- Step A prevents "the doc says one thing, the code says another" drift

## What NOT to do

- Do not expand lemma signatures to reflect cluster structure (Step A principle)
- Do not fill B3 cases in B2 (they need different mechanisms)
- Do not change the dispatch after B1 (it's frozen)
- Do not guess at `simpa` failures — inspect the actual term (see `lean-debugging-inspect-first.md`)
