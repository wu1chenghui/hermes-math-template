# Proof-state verification & anti-drift (long Lean projects)

## Four-source verification protocol (run at every freeze / "closed" claim)
A declaration is "closed" only when ALL of these agree. Used at every phase gate.
1. **LSP diagnostics** — `lean_diagnostic_messages <file> severity=error` → 0 errors;
   a full run → 0 real `sorry`. Sub-second; the fast gate.
2. **`lake build E.Fully.Qualified.Module`** → exit 0, "Build completed successfully
   (N jobs)". Real compile + writes the `.olean`.
3. **`.olean` artifact** — confirm it regenerated:
   `ls .lake/build/lib/lean/<Path>.olean` (NOTE path: `.lake/build/lib/lean/`,
   NOT `.lake/build/lib/`). Fresh timestamp = a real artifact, not an
   in-memory-only check (a module verified only via `lake env lean` has NO `.olean`,
   so a dependent file later fails to build).
4. **`lean_verify <file> <Namespace.theorem>`** → axioms ⊆ {propext,
   Classical.choice, Quot.sound}; ANY `sorryAx` = a reachable sorry (even through
   imports). `axioms=[]` = fully constructive (stronger than clean). This is the
   ONLY reliable "no reachable sorry" check.

Why all four: LSP can pass while there's no `.olean`; `grep` for `sorry` matches
comments/docstrings (false positives) and never catches sorry reached through
imports (false negatives). `lean_verify` axiom-closure is the truth.

## Metadata drift: status docs vs disk
- A status doc (AGENTS.md / a checkpoint) can claim "N sorries / closed / open"
  while the disk differs. Seen: AGENTS said "P3b body = 3 sorries"; disk actually
  had real proofs + 1 unrelated error in one branch.
- A checkpoint can claim "0 errors" while the file has omega errors that the
  previous session never verified. Seen 2026-06-27: checkpoint claimed L1830/L1870
  CLOSED via source-shift, L2106(u>=2) CLOSED via T3 pattern — but (a) L1834/L1853
  had omega failures (i=1 case not handled), (b) L2106 code was never modified
  (sorrie still present). Both were discovered only by LSP diagnostics in the
  NEXT session.
- Lesson: never trust a checkpoint's "closed" or "0 errors" claim without
  running lean_diagnostic_messages(severity=error) on the frontier file.
  Checkpoints are hypotheses; LSP + lake build are ground truth.
- On session start for a long proof project: reconcile the status doc's claims
  against actual LSP diagnostics + a build of the relevant module BEFORE trusting
  the doc. Treat the doc as a hypothesis; LSP/build/lean_verify are ground truth.
- After closing a node, fix the status doc AND any stale docstring immediately so
  the next agent is not misled. Drift compounds.
- Omega specifically: omega errors can survive through "closed" checkpoints
  because they are real errors (build fails) but the previous agent may have
  recorded the checkpoint before a final build verification. Always re-build
  after any checkpoint claim.

## Prove-in-`lean_run_code`-first, land-after (default for non-trivial proofs)
- Write the full proof inside `lean_run_code` (self-contained: `import E.Whatever`
  + `open` + `example`/`lemma`), iterate on errors there (no file mutation), and
  only `patch`/`write_file` to land once it returns success. Produced near-zero-
  rework landings across a whole multi-unit pipeline.
- For **layer-bridging** proofs (Submodule / Subtype / coercions / finite-index
  models) the danger is Lean *interface* (types, defeq, metavars), NOT the math.
  Probe interface points first with tiny `lean_run_code` examples (does the
  statement type-check? does the destructuring work? is this `rfl`?) before writing
  the body. Math being true ≠ the Lean assembling. (See lean-4-proof-writing
  `references/finrank-card-and-interface-bridging.md` for the concrete nuggets.)

## Documentation governance for long multi-doc projects (anti-drift for future agents)
Goal: a fresh agent entering from ANY document converges to the live truth, never
to a superseded result.
- Maintain ONE live-truth doc (e.g. AGENTS.md) that says "read this first" and
  carries current state.
- When a result is superseded/falsified, DO NOT delete the old docs (version-marked
  history — user preference). Add a `DEPRECATED` banner at the very top of each,
  stating what replaced it + pointing to the canonical docs + the live-truth doc.
- Keep a README/index splitting docs into **canonical (live)** vs **deprecated
  (superseded)** lists, with a **new-agent reading order** pointing only at
  canonical + the live-truth doc. A stale README listing the OLD paper as "★ the
  deliverable" is the single worst drift trap — it actively teaches a fresh agent
  the falsified result.
- **Entry-convergence audit**: list every plausible entry door (project root, each
  subdir README, each top-level doc) and confirm each routes to the live truth.
  Fix the doors that don't.
- Batch the banners with an idempotent script (skip if banner already present). Do
  NOT banner generated artifacts (`output/*.tex` etc.) — a markdown banner breaks
  LaTeX; flag them in the index as "generated-from-deprecated" instead.
- `grep` the old claim string (e.g. `dim ... = n`) to find every doc still
  asserting it; cross-check against your canonical/deprecated split so nothing is
  missed.

## Validate the induction INVARIANT's closure (not just the theorem statement)
The skill already says: validate a classification theorem computationally before
proving. New nuance: when the proof is an induction over a reduction/split graph
with an "exception set" (targets allowed to be nonzero), ALSO verify
computationally that the exception set is **closed under the split-children map** —
a non-exception node must only ever produce non-exception children. If it leaks,
the IH won't apply to the child and the induction breaks.
- This session: enumerated all (∉I, off-diag, nonadjacent-source) instances for
  n=6,7,8 → 0 children leaked into I → the non-adjacent induction is
  self-contained, the only irreducible leaf is the adjacent base case. Turned a
  vague "is this doable" into precise risk localization before any Lean.
- The OLD attempt failed because its exception set was too SMALL (it claimed false
  things vanish) → unavoidable sorries. The fix was the CORRECT (closed, minimal)
  exception set, not a better tactic. **When a proof hits an irreducible gap,
  suspect the exception set / statement, not the technique.**

## Phase-gate working style (referee/PM user)
- "Take the green nodes first": land the low-risk *certain* declarations (defs,
  validity lemmas) before the hard math node, so the project keeps moving even if
  the hard node stalls.
- Freeze at stable theorem nodes; offer freeze proactively at the boundary
  (snapshot backup + record the CLOSED milestone + name the next entry point).
- Paper-audit / design `.md` BEFORE Lean for a new proof object; write the design
  in a SEPARATE doc, not the global theory doc.
