# Session Pickup / Project Orientation Recon

When the user says "familiarize yourself with the project" or you're starting a
fresh session on the `e/` formalization (`/opt/lean-home/lean-projects/e/`),
run this ordered recipe to get oriented fast and verified — NOT a file-by-file
read of all 60+ `.lean` files. Familiarization ≠ authorization: produce a
briefing and wait for direction; do not start editing.

## The recipe (parallelize independent steps)

1. **Load both Lean skills in parallel**: `lean-4-workflow` +
   `lean-4-proof-writing`. (Mandatory — they carry the architecture, the
   anti-drift discipline, and the sparse-coeff tactics.)

2. **Get a CLEAN source-tree listing.** `search_files target=files pattern=*.lean`
   on the project **root** floods with hundreds of `.lake/build/...` artifacts
   (`.olean`, `.c`, `.trace`, `.ilean`, `.hash`, `.setup.json`) — `.lake/` is a
   sibling of `E/`. **Narrow the path to the source subdir**:
   `search_files(target=files, pattern=*.lean, path=…/e/E)` → ~67 clean source
   files, no build noise. Same trap if you ever list the root for the docs.

3. **Read `AGENTS.md` head + tail, not the middle first.**
   - HEAD = the `🔒 LOCKED` classification theorem (immutable math: dim = n+5,
     ker Φ = n+4, ideal I, Φ definition). This is the canonical spec.
   - TAIL = the live frontier. The user's working discipline appends a new
     `🔒 SESSION CHECKPOINT` block + a `⬜ NEXT SESSION ENTRY` at the BOTTOM after
     every phase-gate closes. So the last few hundred lines hold the current
     unit table, what's CLOSED, and the exact next node — read the tail to find
     where work actually stands. The middle is mostly historical chronology.
   - **If the user pasted a "Current frontier (resume point)" block** (a recurring
     habit), treat it as their CLAIM of the tail state, NOT ground truth —
     cross-check it against the AGENTS tail checkpoints it cites + the LSP
     frontier-file diagnostic, then report "confirmed" or "relocated". (Observed
     2026-06-25: a recent same-day multi-checkpoint tail (d–h) matched disk
     exactly — GATE-A/GATE-B closed, 0 error / 0 sorry, only the known
     `CharNeTwo` false-friend warnings. The verify still runs; it just confirmed
     rather than relocated.)

4. **`lean_file_outline` the frontier file** named in `NEXT SESSION ENTRY`
   (token-efficient — imports + decl signatures, no bodies).
   **⚠ Pitfall:** `lean_file_outline` can time out (120s) on files >1500 lines
   (observed: ImageContainment.lean at 2048 lines). Fallback: use `read_file` with
   `limit=80` on the frontier file to see imports + first declarations, then
   `lean_diagnostic_messages(severity='error')` for ground-truth error list.

5. **Ground-truth with `lean_diagnostic_messages` + `search_files` on the frontier file.**
   AGENTS.md can lag disk (observed: a "3 sorries" note that was already closed).
   The real error/sorry list RELOCATES the true frontier. **Pitfall:**
   `lean_diagnostic_messages(severity='error')` catches compile errors but NOT
   sorries — Lean 4 treats `sorry` as a proof-stuck marker, not an LSP error.
   To count sorries, use `search_files(pattern='sorry', target='content',
   path=<frontier_dir>)` then filter OUT docstring mentions (lines where `sorry`
   appears inside a `/-` comment block or after `--`). For the frontier file
   whose error count is zero: a `search_files` `sorry` hit inside a proof body =
   a real live sorrie. Benign `unusedSectionVars [CharNeTwo F]` warnings are
   the known false-friend (see SKILL.md pitfall) — NOT a defect, do not "fix"
   them. For a deeper closure check use `lean_verify <thm>`
   (axioms ⊆ {propext, Classical.choice, Quot.sound}, no `sorryAx`).

## Load-bearing operational facts to surface in the briefing

- **`lake build` compiles ONLY `E.lean`'s import tree = the LEGACY dim=n graph.**
  The live `dim = n+5` pipeline (IdealI, PhiOperator, WidthFiltration,
  BasisCocycles, RepresentationBridge, DimensionTheorem, Spanning,
  ImageContainment) is NOT in that tree. Build it by fully-qualified name:
  `lake build E.Classification.Spanning` (deps compile transitively).
- **Active vs frozen.** ACTIVE = the `E/Classification/*` n+5 chain
  (WidthFiltration, BasisCocycles, ImageContainment, RepresentationBridge,
  Spanning, DimensionTheorem) **plus its `E/Core/` foundation**, the full chain
  MatrixBasis → BracketFormula → ProjectionIdentity → HalfDerivation →
  HalfProjection → IdealI → PhiOperator. FROZEN = everything else
  (`E/HalfDerivation/*`, the legacy NnDerivation chain, the many `E/` root
  analysis files, the legacy Reduction→Evaluation→Leaf→HalfClassification dim=n
  DAG) — do not touch when working the n+5 pipeline.
  - ⚠ **`E/Core/` name-collision trap** (easy to misclassify, cost-free to
    avoid): the ACTIVE `ProjectionIdentity.lean` + `HalfProjection.lean` are NOT
    the FROZEN `Projection.lean` + `Coefficient.lean` + `Derivation.lean` (the
    legacy NnDerivation chain). The near-identical names sit side-by-side in the
    same directory. `coeffOf_f` (the `coeffOf ↔ D.coeff` bridge the n+5 leaf
    lemmas lean on) and the projection identities come from the *Identity* /
    *Half* files — never from the bare `Projection`/`Coefficient`. Confirm via
    the frontier file's actual `import` list (e.g. ImageContainment imports
    `E.Core.HalfDerivation`, `E.Core.PhiOperator`), not by name guessing.
- **Two-layer FROZEN model** (version-marked, old not deleted): old dim=n =
  historically correct but SUPERSEDED (falsified by T(E_12)=E_25 in N_5); new
  dim=n+5 = active, in integration/wiring phase (math layer sealed).
- ⚠ **ImageContainment is an ORPHAN module (2026-06-27).** No file imports
  `ImageContainment.lean`. The live pipeline (RepresentationBridge → Spanning
  → DimensionTheorem) works entirely without it. `imageContainment_skeleton`
  has ZERO callers. D3 (HalfDer = F·Id ⊕ kerΦ) is NOT YET WIRED. The
  skeleton's induction (target-width, per-target case split) diverges from
  the paper's approach (source-width, CE-generation lemma). See
  `references/ce-generation-vs-skeleton-divergence.md`.

## Output shape

A structured briefing: (1) project + theorem + environment, (2) the loaded
skills, (3) active-vs-frozen file map, (4) VERIFIED current state + the precise
frontier node, (5) paper side (`/workspace/proof-reconstruction/`, canonical =
FINAL-THEOREM.md + WIDTH-COLLAPSE.md). Then ask for direction. Keep it scannable;
the user dislikes progress spam.
