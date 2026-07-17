# DAG Module Audit — Checking Whether a Sorrie Is Actually Needed

## When to use

When a sorrie has resisted multiple rounds of local search (partner audit,
source-shift, SCC, structural audit), BEFORE investing more time, check whether
the enclosing lemma/module is even connected to the downstream proof pipeline.

## The methodology

1. **Check lemma callers.** Search for the lemma name across the entire project:
   ```
   search_files(pattern='<lemma_name>', target='content')
   ```
   If only the definition line matches → ZERO callers → dead branch.

2. **Check module importers.** Search for imports of the enclosing module:
   ```
   search_files(pattern='import.*<ModuleName>', target='content')
   ```
   If no files import it → the ENTIRE module is orphaned.

3. **Trace the live pipeline.** Identify which modules form the actual downstream
   chain (finrank → spanning → representation bridge → ...), and verify each
   step has the imports it needs.

4. **Compare to the blueprint.** The AGENTS.md or proof-reconstruction docs may
   describe an intended architecture that differs from the actual import graph.
   The import graph is ground truth.

## Example: ImageContainment orphan discovery (2026-06-27)

The `imageContainment_skeleton` theorem had a stubborn sorrie (L2099, first-row
rectangular coefficient). After exhaustive structural audit proved the coefficient
is invisible to ALL single-equation mechanisms, a module-level DAG audit revealed:

- `imageContainment_skeleton`: 0 callers (only the definition line matches)
- `ImageContainment.lean`: 0 importers (no file imports it)
- Live pipeline: RepresentationBridge → Spanning → DimensionTheorem — NONE import
  ImageContainment
- D3 (HalfDer = F·Id ⊕ kerΦ) is the only thing that WOULD need image containment,
  but D3 isn't wired yet

**Result**: L2099 is a dead branch in a not-yet-wired module. The sorrie cannot
be a blocker for the current pipeline. This finding redirected effort from local
proof search to architectural assessment.

## When the result is "dead branch"

- Annotate the sorrie clearly with the DAG findings
- Do NOT delete the code (it may be needed when D3 is wired)
- Move on to sorries that ARE in the live pipeline
- File the finding in AGENTS.md checkpoint
