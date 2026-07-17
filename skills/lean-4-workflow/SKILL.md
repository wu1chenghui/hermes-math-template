---
name: lean-4-workflow
description: >-
  Set up, maintain, and do Lean 4 work (theorem proving, proof verification)
  inside a WSL+Docker container with 9p filesystem constraints and unreliable
  GitHub connectivity. Covers the Lean toolchain (elan, lake, mathlib),
  named-volume migration, permission patterns, offline package setup from
  ZIP archives, creating/verifying Lean projects, AND the methodology for
  reconstructing human-readable paper proofs from Lean formalizations.
  See `references/proof-reconstruction-methodology.md` for the full pipeline.
  Web search debugging: `references/web-search-debugging.md`.
  this skill is about enabling Hermes itself to do Lean work.
---

# Lean 4 Workflow

> **Communication language (this user):** Chinese for discussion, strategy, and
> project navigation; English for Lean code bodies, lemma names, and tactic scripts.
> The user profile says "Bilingual (Chinese strategy, English Lean code)" — follow
> this split consistently. Briefings, checkpoint reports, and error analysis in
> Chinese; Lean source stays in English.
>
> **Precision in claims:** The user strongly prefers precise formulations over
> overstated ones. Distinguish "natural basis" from "unique basis"; "structural
> explanation" from "second proof"; "the count 5 has a clear structural origin"
> from "any classification must give exactly 5." When stating confidence, use
> "substantially increased confidence" not "probability is negligible." Do NOT
> claim "100% confirmed" without a clean build verification.
>
> **Mathematical Trust Audit methodology:** When the user asks to verify that a
> Lean formalization is mathematically correct (not just that it compiles), use
> the layered audit taxonomy in `references/audit-taxonomy.md`. The taxonomy
> progresses from engineering audits (clean build, dead modules) through proof
> structure (theorem dependency, hidden assumptions) to mathematical understanding
> (origin, derivation, invariant identification). Stop adding audits when the
> user signals they want to move from "verifying Lean" to "understanding the
> mathematics."
>
> **For attack-style mathematical auditing** — when the user wants to stress-test
> a completed proof rather than just verify it compiles: use the methodology in
> `references/mathematical-auditing-methodology.md`. It covers counterexample
> construction, constraint coverage matrices, rank certificates, dependency graph
> connectivity, and the dim=n failure archetype. Core principle: **attack, don't
> explain** — every failed counterexample increases confidence more than any
> explanatory audit. Prefer mathematical certificates (equation incidence matrices,
> Jacobian rank proofs, dependency graphs) over prose summaries.
>
> **Documentation precision (AGENTS.md, checkpoints, bug reports):** Do NOT overstate
> technical claims. Prefer precise formulations: "instance projection definitional
> equality not automatically exposed" over "implicit binders break rfl"; "elaboration
> context differs between lean_run_code and file compilation" over "kernel differs."
> When a skeleton comment is found incorrect, record exactly WHY (with Lean output or
> computational evidence), not just that it "is wrong."

> **AGENTS.md editing protocol (ADD-ONLY, PROPOSE-FIRST).**
> AGENTS.md is the project's cumulative session log. NEVER delete or replace
> existing content — only ADD new sections. Structure: keep the intro + doc
> index intact, insert new content between the header separator and the
> CLASSIFICATION THEOREM section, and leave all session history untouched.
> Before editing: (1) propose the change to the user, (2) wait for confirmation,
> (3) execute. If a mistake is made, restore from `/workspace/backups/` before
> attempting any further edits. The backup directory contains standalone
> `AGENTS_pre_update.md` / `AGENTS_post_update.md` files AND timestamped
> `e_pre_*` / `e_post_*` tar.gz snapshots of the full project. See
> `references/backup-system.md` for the restore procedure and naming conventions.

> **Verification & anti-drift (read when doing real proof work):**
> `references/proof-state-verification-and-anti-drift.md` — the 4-source "closed"
> protocol (LSP diag + `lake build` + `.olean` timestamp + `lean_verify`
> axiom-closure), reconciling AGENTS-style status docs against disk (they drift —
> e.g. "3 sorries" that were actually closed), prove-in-`lean_run_code`-first then
> land, documentation governance (canonical/deprecated banners + new-agent
> reading-order convergence audit), checking an induction's exception-set is CLOSED
> before proving, and the phase-gate / green-nodes-first working style.
>
> **Pitfall: replacing a lemma that has callers.** When adding a new, cleaner lemma
that supersedes an existing one (e.g. an hI-free local SCC replacing an hI-dependent
boundary lemma), do NOT delete the old lemma until all callers are migrated. First add
the new lemma alongside the old one, verify it compiles, then migrate callers one by
one, then delete the old lemma. Deleting prematurely breaks downstream callers (e.g.
`transfer_rep_A` calling the still-needed `boundary_rep_A`).

**`bracket_zero` argument count.** `HalfDerivation.bracket_zero` takes exactly 11 proof
arguments after the 6 indices (hi, hij, hjn, hc, hcd, hdn, hu, huv, hvn, hj_ne_c,
hi_ne_d). Use `(by omega)` for all of them unless the index is not bounded by omega
(e.g. `u < v` with specific values). A wrong argument count gives a cryptic "Function
expected" or "Application type mismatch" error.

**`bracketIdentity_eq_expanded` sign awareness.** The expansion is `T1 - T2 + T3 - T4`.
T2 and T4 have minus signs. When both fire, the equation is `-T2 - T4 = 0`, NOT
`T2 + T4 = 0`. Use `linear_combination -hb` to flip signs when needed, or adjust the
statement to match the raw expansion (e.g. `-y - z = 0`), then derive the positive
form separately.

**Pitfall: forward reference in same file.** Lean processes top-to-bottom within a
`.lean` file. If lemma A uses lemma B, B must be defined before A — even in the same
file. Check dependencies transitively before moving.

**Code relocation safety (when moving lemma B before lemma A to fix a forward reference):**
1. **Use Python, not manual editing.** Read lines via Python, extract the block,
   delete from original position, insert at target. Manual cut/paste introduces
   off-by-one errors in large files (2000+ lines).
2. **Check docstring integrity after move.** A `/--` doc comment without a closing
   `-/` will silently consume the next declaration's docstring or cause
   `unterminated comment` / `unexpected token '/--'` errors. Count that every
   inserted docstring has exactly one `-/`. Also check that a docstring wasn't
   split: if the old code had a multi-paragraph docstring and you only moved the
   second half, the first half becomes an orphan (closed `-/` with no declaration
   attached → `unexpected token` error) and the second half opens without a `/--`
   starter. Fix by either making each half a complete standalone docstring, or
   deleting the orphaned half.
3. **Check definitional equality after `rw`.** `rw [hi_eq_n1]` where `hi_eq_n1 : i = n-1`
   changes `coeffOf D i n u n` to `coeffOf D (n-1) n u n` — but if the goal was
   `coeffOf D i (i+1) u (i+1)` (after `hvi : i+1 = v`), you also get
   `(n-1)+1` which is NOT definitionally `n`. Use
   `rw [hi_eq_n1, show (n-1 : ℕ)+1 = n by omega]` to handle both.
4. **Check `linear_combination` variable alignment.** `linear_combination h_eq` fails
   with `ring failed, ring expressions not equal` when `h_eq` mentions variable `X`
   but the goal mentions variable `Y` where `X=Y`. Fix: `rw [← hX_eq_Y]` (or forward)
   to align variables before calling `linear_combination`. The `rw [hX_eq_Y] at h_eq`
   pattern is a no-op if `h_eq` doesn't contain `X` — use `rw [← hX_eq_Y] at h_eq`
   to rewrite `Y`→`X` in the hypothesis instead, or `rw [hX_eq_Y]` on the goal.

**Pitfall: Structural transference mistaken for invisibility.** When occurrence audit shows
a coefficient never appears in local half-Leibniz equations, it may not be "invisible" —
it may be *transferred* to a different coefficient type. In the v=n case,
`coeff(i,i+1;i,n)` = GOAL was transferred to wide-source b-channel coefficients
`coeff(1,i+1;1,n)` and `coeff(2,i+1;2,n)` via bracketSource. The GOAL was not invisible;
it was expressed in terms of other coefficients that were then forced to zero by a
separate constraint. Before concluding "no local proof exists", check whether the
coefficient appears indirectly through bracketSource (not just bracketIdentity).

**Methodology: computational-before-formal.** When a mathematical gap resists
all standard proof techniques, invest 30 minutes in computational experiments
before spending days in Lean. Build the linear system over ℚ, compute the
nullspace, and check: (a) whether the theorem is true, (b) which equations
collectively force the result. This converts "is there a proof?" into "where
is the proof?" — a much easier question. See
`references/computational-experimentation.md`.

**Pitfall: `width_b_zero` requires `I_filtered`.** The lemma `width_b_zero` in
`WidthFiltration.lean` takes `hI : I_filtered D.coeff hn` as a hypothesis. This
creates a circular dependency when trying to prove `I_filtered` for centered
half-derivations — you cannot use width_b_zero to prove what width_b_zero needs.
The Boundary Rigidity proof (`references/boundary-rigidity-proof.md`) bypasses
this entirely by using only pure half-Leibniz equations.
`BracketFormula.lean` is `(if j=c then coeff i d u v else 0) - (if i=d then coeff c j u v else 0)`.
It gives coeff(i,d;u,v) when source_col=partner_row, or coeff(c,j;u,v) when
source_row=partner_col. It does NOT equal coeff(src; target_row, partner_row) in general.
When sources commute (bracket_zero), bracketSource = 0, and the half-Leibniz equation
reduces to bracketIdentity = 0. Do not assume bracketSource exposes GOAL directly —
verify the j=c and i=d conditions explicitly.

**Pitfall: bracket_zero uses commuting sources that must be DIFFERENT.**
When source and partner are identical (e.g. source (1,2), partner (1,2) at i=1),
bracket_zero is still valid ([E_ij, E_ij]=0), but bracketIdentity expansion gives
T2 and T3 both firing (same coefficients, opposite signs), producing a tautology
0=0 that carries no information about GOAL. Always check i=1 degeneracy when using
source-shift with source (1, u) and partner (i, i+1).

**Pitfall: `subst` causes `MatIdx` type inference pollution.** In deep contexts
with `open MatIdx`, `subst heq` can cause `i` and `n` to be interpreted as
`MatIdx → ℕ` functions rather than `ℕ`, giving cryptic errors about implicit
arguments `α`, `β`, `γ`. Prefer `rw [heq]` over `subst heq` when this occurs,
especially after other substitutions have already been applied.

**For the kerPhi_lift pattern** — lifting a coefficient-table Submodule element into a Prop-guarded structure when the Submodule's membership predicate matches the structure's field conditions. See:
  `references/kerphi-lift-pattern.md`.

**For D4 assembly methodology** — when entering the final assembly phase (all local mathematics closed, remaining work is structure-level plumbing): the **canonical approach that succeeded** uses `coeffOf_map : HalfDerivation →ₗ[F] V F` (a linear map sending D to coeffOf D, which zeros invalid indices). Then compute `finrank` on the **range** of this map: `range(coeffOf_map) = span{coeffOf(Id)} ⊔ kerΦ`, disjointness gives `1 + (n+4) = n+5`. This bypasses the older plan of building an `Equiv`/`LinearEquiv` between `HalfDerivation` and `F×kerΦ` — that approach ran into the problem that `HalfDerivation` has no `Module` instance (invalid-index junk makes its raw `finrank` ill-defined). The `coeffOf_map` route is cleaner because it works on the valid-index canonical image. See: `references/d4-coeffOf-map-pattern.md`.

**Key DAG (OBSOLETE — kept for historical reference only):**
```
kerPhi_lift → coeff_injective → decompose → halfDer_decomposition (Equiv) → finrank
```
where `halfDer_decomposition` is a plain `Equiv` (not `LinearEquiv`) between `HalfDerivation` and `F × kerΦ`, and `finrank_halfDer` uses `Nat.card` or a separate basis argument on `kerΦ`'s already-proven `finrank`. See:
  `references/kerphi-lift-pattern.md`.

**Pitfall: `open MatIdx` namespace clash — FIX FIRST, don't work around.** When `open MatIdx` is in scope, any variable named `i` gets resolved as `MatIdx.i` (function `MatIdx → ℕ`) rather than `ℕ`. Diagnostic: `@Eq (MatIdx ?m → ℕ) i 1`. Before attempting `convert`, `rw`-avoidance, or other workarounds: FIRST search the file for actual `MatIdx.X` usage. If zero uses outside the `open` line itself, REMOVE `open MatIdx` — this fixes ALL elaboration issues simultaneously. Only resort to workarounds when the file genuinely uses MatIdx fields. See:
  `references/kerphi-lift-pattern.md`.

**Pitfall: `subst` can DESTROY section variables (notably `n`).** When a
hypothesis `h : v = n` is used with `subst h`, Lean removes `v` but can
also clear the section variable `n` from the context, causing downstream
errors like "Unknown identifier `n`" at calls to lemmas that use `n` as
an implicit binder (e.g. `boundary_rigidity (F := F) (n := n)`).
**Prefer `rw [heq]` over `subst heq`** — keep the hypothesis as an
equality and rewrite it explicitly where needed. This is especially
critical when `n` is used in lemma calls, `coeffOf_f` bounds, or
as a section variable imported from other files.  **Concrete example:**
in the `v=n` branch of `centered_image_in_I`, `subst hvn_eq` where
`hvn_eq : v = n` immediately produces "Unknown identifier `n`" at
`hcoeff_n : coeffOf D' i j u n = …` — every occurrence of `n` becomes
unresolvable. Use `rw [hvn_eq]` throughout instead.

**Pitfall: `induction' … generalizing i j` + full `revert` scrambles argument order.**
When using `induction'` with `generalizing`, the generalized variables drag ALL
context hypotheses that mention them into the induction goal BEFORE the reverted
hypotheses. If you `revert hi hij hjn h_off h_ge2 hne` AND use `generalizing i j`,
the `ih` will have 15+ arguments in unpredictable order. **Rule: either `revert`
everything and DROP `generalizing`, OR revert one thing and use `generalizing`.**
Full details + the `Phi` universal-quantifier trap at:
  `references/induction-generalizing-pitfall.md`

**Pitfall: `simp [I_target_set hn]` fails.** `I_target_set hn` is a `Set (ℕ × ℕ)`,
not a rewrite lemma. `simp` will report "Invalid simp theorem: Expected a
proposition, but found". Use one of these patterns instead:
  - `apply h_notI; rw [heq]; simp [I_target_set]`
  - `show (1, n-1) ∈ I_target_set hn from by simp [I_target_set]`
Note: `simp [I_target_set]` WITHOUT `hn` works — the `hn` argument is what
causes the type error.

**Pitfall: `open MatIdx` causes namespace clash when rewriting `ℕ` literals.**
Files with `open MatIdx` expose a `MatIdx.i` field that competes with binder
`i : ℕ` and causes `simp` / `rw` on `ℕ` expressions to produce cryptic errors:
```
don't know how to synthesize implicit argument `α`
  @Eq (MatIdx ?m.716 → ℕ) i 1
```
This occurs when `simp [I_target_set, hn3]` tries to simplify `(1, n) ∈ I_target_set hn3`
— the literal `1` is being interpreted as `MatIdx → ℕ` type instead of `ℕ`.
**Workaround**: avoid `rw [hi1]` or `simpa [hi1, ...]` in `open MatIdx` contexts.
Instead, use `convert` with explicit type ascription or move the rewrite inside a
`have` block where namespace resolution is local:
```lean
have hmem : (1, n) ∈ I_target_set hn3 := by
  unfold I_target_set; simp
convert hmem using 1; exact hi1.symm
```
Or better: avoid `open MatIdx` in new files and use explicit `MatIdx.E` etc.

**Pitfall: searching for GOAL in the wrong target column.** The T1 term in
`bracketIdentity_eq_expanded` is `coeff(i,j; u, c)` where `c` is the PARTNER's
row index, NOT the target column. GOAL = `coeff(i,i+1; 1, v)` can appear as T1
in equations with target column `v+1` (or any `d > v`), as long as the partner
has row `v`. When auditing "does GOAL appear in any bracketIdentity?", enumerate
across ALL target columns, not just the one matching GOAL's column.

**Pitfall: T3 in `bracketIdentity_eq_expanded` fires more often than expected.**
T3 condition is `u=i ∧ j<v`. When target row equals source row and target col >
source col (common in width_a/c chain proofs), T3 ALWAYS fires, introducing a
partner-side coefficient `coeff(partner_col, target_col; source_col, target_col)`.
The code's \"only T1 fires\" assumption is wrong. Handle T3 explicitly or convert
to a sorry.  See `references/bracketidentity-t3-pitfall.md` and
`references/lsp-error-fix-patterns.md#t3-in-bracketidentity-always-fires-for-same-row-targets`.

**For LSP error fix recipes** (coeff-vs-coeffOf direction, subst variable capture,
Nat subtraction definitional equality, simp Ne symmetry, errors-before-sorries
workflow): see
  `references/lsp-error-fix-patterns.md`.

**For Meta-Proof Object Audit (Phase 0)** — the FOUNDATIONAL audit that must precede all other audits. Defines every mathematical object in the proof system (5-layer pyramid: Pair → Vector Identity → Scalar Identity → Functional → Constraint), establishes EqSpace as a vector space, and prevents the dim=n class of errors (object-level conflation causing missed constraint classes). When the user asks "what are we actually proving" or critiques audits as "text summaries not certificates," load this FIRST. See:
  `references/meta-proof-object-audit.md`.

**For the project architecture audit methodology** (clean-build verification, import DAG vs proof graph, theorem dependency audit, documentation freeze, and D4 debugging lessons including "Max Type" misdirection and `LinearEquiv.finiteDimensional` direction): see
  `references/project-architecture-audit.md`.

**Session pickup / project orientation (read at the START of a session):**
> `references/project-pickup-recon.md` — the ordered recon recipe to get oriented
> fast on the `e/` formalization without reading all 60+ files: clean source-tree
> listing that avoids the `.lake/build` artifact flood (narrow `search_files` to
> the `E/` subdir, not the project root), AGENTS.md head=locked-theorem /
> tail=live-frontier+`NEXT SESSION ENTRY` reading order, frontier-file LSP
> ground-truth, and the `lake build` = legacy-dim=n-graph-only gotcha (the live
> dim=n+5 pipeline builds only by fully-qualified module name).

**Pitfall: `search_files` with glob may miss files under deep paths.**
`search_files(pattern='*.lean', target='files', path='/opt/lean-home/lean-projects/e/E')`
can return 0 results even when 70 `.lean` files exist.  Use `execute_code` with
`os.walk` for reliable source-tree listing, filtering out `.lake/` dirs.

**Pitfall: AGENTS.md is 3500+ lines.**  A single `read_file` call times out.
Read in chunks: start with `offset=1, limit=200` for the theorem statement,
then paginate with `offset=200, limit=400`, `offset=600, limit=400`, etc.,
or use `execute_code` with `read_file` tool to target specific checkpoint
sections.  The NEXT SESSION ENTRY / latest checkpoint is always near the end.

**For consistency-auditing theorem statements against concrete counterexamples before filling sorries** (stop depth-first engineering; verify the statement domain first): see
  `references/consistency-audit-before-engineering.md`.

**For reverse dependency audit** — when a coefficient resists all local proof, search the ENTIRE project for any theorem whose CONCLUSION already states what you need (not proof-body occurrences; not callers). Distinct from occurrence audit and DAG module audit. Catches the L2136 pattern (math proven, DAG disconnected). See:
  `references/reverse-dependency-audit.md`.

**Pitfall: v=n bracketSource→diagonal mechanism is insufficient.** ImageContainment.lean:2072-2081 claims that for centered D', partner (i+1,n) couples GOAL = coeff(i,i+1; i,n) to diagonal coeff(i,n; i,n) via bracketSource. Manual verification: bracketSource = coeff(i,n; i,n) = 0 (centered), bracketIdentity also vanishes (T1 = coeff(i,i+1; i,i+1) = 0, T3 = coeff(i+1,n; i+1,n) = 0), giving 0=0 with NO constraint on GOAL. The comment is incorrect. The ACTUAL proof (discovered 2026-06-28) is a 3-equation syzygy: Eq_A (src=(1,i), ptr=(i,i+1), tgt=(1,n)) + Eq_B (src=(1,2), ptr=(2,i+1), tgt=(1,n)) + Eq_C (src=(2,i), ptr=(i,i+1), tgt=(2,n)) combine to give g = 2a - 2a + 2b - 2b + g = g, i.e. g = 2a where a = coeff(1,i+1;1,n). Three pure half-Leibniz equations, no I_filtered. See `references/proof-extraction-from-constraint-rref.md`.

**For SCC extraction before proof search** (when you have one equation and can't find a second independent relation — the SCC is larger than 2×2; stop searching for partners and extract the full dependency graph first): see
  `references/scc-extraction-methodology.md`.

**For the local-SCC-over-width-chain pattern** — when a blueprint calls for width-induction chains but the actual dispatch uses self-contained 3×3 local systems (boundary_local_234, boundary_local_right, boundary_coupling, etc.), plus the orphaned-lemma annotation convention and dependency-audit methodology: see
  `references/image-containment-local-scc-pattern.md`.

**For D3 skeleton-phase implementation patterns** (set-vs-obtain, bracket_zero arg order, Nat-subtraction omega avoidance, ZERO proof template): see
  `references/d3-skeleton-implementation-patterns.md`.

**For D3 assembly methodology** — when to stop drilling into individual sorries and instead write the full proof skeleton first: the three-outcome classification (A/B/C), the `ext_coeff` pattern for `HalfDerivation` equality, and the `HalfDerivation` typeclass pitfall (standalone instances without bundled `AddGroup`). See
  `references/d3-assembly-methodology.md`.

**For D3 chain induction with `Nat.decreasingInduction`** (dependent-motive recipe, `rw at *` pitfall, `set` let-in gotcha, rebuild-from-scratch pattern): see
  `references/d3-chain-induction-pattern.md`.

**For the TRANSFER dispatch mechanism analysis** — proof that for adjacent source
(i,i+1) with u<i, v≠i+1, no valid partner exists whose bracketIdentity involves
the GOAL coefficient. The constraint must come from a two-step chain via
bracketSource → nonadjacent coefficient → coeffOf_source_exists + IH.
See `references/transfer-dispatch-mechanism-analysis.md`.

**For the local-SCC closure patterns** (3×3 det=1, 3×3 det=3, 2×2 bracketIdentity+coeffOf_cond):
  see `references/local-scc-pattern-catalog.md`.

**For the A-fire 3×3 SCC mechanism** (A = coeff(k,k+1;1,k+1), B = coeff(k,k+2;1,k+2),\
C = coeff(k,k+3;1,k+3); three equations → det=2, characteristic-independent closure;\
NOT a σ-mirror of internal_det_step; do NOT use exponential chains): see\
  `references/a-fire-scc-mechanism.md`.

**For vertical adjacent coefficient structural analysis** — exhaustive delta-probe
enumeration to map every bracketIdentity equation involving a given coefficient,
coupling-chain layering, and the rule "probe first, prove second" (do NOT change
induction predicates until the coupling structure is exhaustively mapped): see
  `references/vertical-adjacent-structural-analysis.md`.

**For the C-fire width-2 k=2 local 2×2 closure** — three-equation self-contained
system (coeffOf_cond at two levels + half-Leibniz T2), no hI/IH/chain needed,
`if_pos`/`if_neg` pattern for ∧-conditions, `congrArg Neg.neg` solver, and the
σ-mirror pattern for width_a_chain: see
  `references/c-fire-local-2x2-closure.md`.

**For the T3 cross-source elimination pattern** — when a coefficient is invisible to half-Leibniz for its own source, use a different source where it appears as T3 (source (1, target_row), partner (i, i+1), target (1, n)): see
  `references/t3-cross-source-elimination.md`.

**For the Max Type pitfall on V=ℕ⁴→F** (REFUTED 2026-06-30 — the error was a `LinearEquiv.finiteDimensional` direction mistake, NOT a fundamental typeclass issue; see the corrected pitfall in this SKILL.md and `references/finrank-typeclass-conflict.md`): see `references/max-type-finite-dims-pitfall.md` (now marked as refuted).

**For IsValidSupp in kerΦ — use coeffOf as canonical representative** (when a HalfDerivation's raw `.coeff` may have garbage at invalid indices, `coeffOf D` is the normalized version: returns 0 at invalid indices by construction, equals `D.coeff` at valid indices via `coeffOf_f`.  Use `coeffOf D` for kerΦ membership — IsValidSupp is automatic.  The Phi bridge from `coeffOf D'` back to `D'.coeff` is handled by proving bridges for the primitives bracketSource / bracketLeft / bracketRight (2 conditions each, 4 branches), then assembling bracketIdentity and Phi in one line each.  See `references/coeffOf-bridge-pattern.md`.)

**Pitfall: `dim Der(N_n) = 2n-1` is WRONG for strictly upper triangular N_n.**
The correct formula is dim Der(N_n) = n(n-1)/2 + 2n - 3 (Leger--Luks).
The 2n-1 number belongs to a different algebra. The half-derivation paper
does not depend on this formula, but introductory comparisons or lecture
notes must use the correct one. When citing "well-known" results, verify
against primary sources — don't propagate errors from memory.

**Pitfall: `lake clean` is catastrophic for mathlib projects.**  Running `lake clean`
deletes ALL `.olean` files including mathlib's ~2900 modules.  Rebuilding from scratch
takes 30–60 minutes.  NEVER run `lake clean` unless you explicitly need a full rebuild
and have the time budget.  Prefer `lake build <Module>` to test specific modules;
it only rebuilds changed dependencies.  If you suspect stale cache, `touch` the
suspicious `.lean` file and rebuild just that module.

**Pitfall: stale olean caches mask compile errors.**  `lake build` can report
"0 errors" for a module that has NEVER actually compiled with the current source,
because it reuses a cached `.olean` from before the changes.  Symptoms:
- `lake build E.Classification.Centering` → 0 errors, finishes in <1 second
- `lake clean && lake build E.Classification.Centering` → 8 errors

The root cause: no `.lean` file was touched or modified, so lake assumes all
`.olean` files are up-to-date and skips recompilation.  **To verify a specific
module from scratch:** delete its `.olean` (rm the `.lake/build/lib/.../<Module>.olean`)
or run `touch <Module>.lean && lake build <Module>`, THEN rebuild.
**`lake build` may silently succeed on stale cache too.** If a module was compiled
once, subsequent `lake build` reuses the old `.olean` without recompiling, even
if the source was later edited. A green build does NOT prove the current source
compiles.
**CRITICAL: AGENTS.md build-status claims may be unreliable.** A "lake build = 0 errors"
claim in documentation may have been made against stale olean cache.
**Only `lake clean && lake build` is fully trustable.**

**Pitfall: "Max Type" error is often misleading.** The error message "failed to
synthesize instance of type class Max Type" can be a DOWNSTREAM artifact of
typeclass elaboration failure in a specific syntactic position, NOT a universe
problem. Before concluding "Max Type = universe issue," test whether the same
expression works in a different position (goal vs `have` type vs `let` binding).
In one case, `Module.finrank F (sId ⊔ kerPhi ...)` as the type of a `have`
statement triggered Max Type, but the same expression worked fine as a goal
after `rw`. Moving the proof inline (directly on the goal, not wrapped in
`have`) avoided the elaboration path entirely.

**Pitfall: `LinearEquiv.finiteDimensional` direction is DOMAIN → CODOMAIN.**
When constructing `FiniteDimensional` instances for Submodules via `LinearEquiv`:
`f.finiteDimensional` proves `FiniteDimensional K (codomain)` from
`FiniteDimensional K (domain)`.  `f.symm.finiteDimensional` goes the opposite
direction.  If your `LinearEquiv` maps `F` (trivially FD) to `sId`, use
`h_equiv.finiteDimensional inferInstance` — NOT `.symm`.  Using `.symm`
causes `Submodule.finrank_sup_add_finrank_inf_eq` to fail with a misleading
"Max Type" error (the real problem is missing `FiniteDimensional` instances).
See `references/finrank-typeclass-conflict.md` (updated 2026-06-30).

**Pitfall: "Max Type" on finrank lemmas is often a MISDIAGNOSIS.**  The 2026-06-30
investigation proved that the "Max Type" error at `Submodule.finrank_sup_add_finrank_inf_eq`
in Centering.lean was NOT a universe issue — it was a cascade from a
`LinearEquiv.finiteDimensional` direction error that left `FiniteDimensional`
instances unsynthesized.  Before concluding "Max Type", verify that ALL
`FiniteDimensional` instance arguments are being provided in the correct direction.
See `references/finrank-typeclass-conflict.md` and `references/mathlib-finrank-sup-compatibility.md`.

**Pitfall: `simp` on Subtype elements is ineffective.**  `simp [v]` where `v : sId`
(Subtype of a Pi type) makes no progress.  Use explicit helper lemmas
(e.g., `have hpos : coeffOf ... 1 2 1 2 = (1 : F) := ...`) and `simpa` with
those lemmas.  A bare `simp [v] at this` is dead code.

**Pitfall: `Module.finrank F (sId ⊔ kerPhi ...)` as a `have` hypothesis type triggers "Max Type".**  
When `Module.finrank F (submodule_sup ... ) = n+5` appears as the TYPE of a
`have h_fin : ... := ...` statement, Lean's type elaboration on `Submodule` sup
can trigger typeclass search that fails with "Max Type" — even though the SAME
expression works fine as a goal type (introduced by `rw`).  

**Fix**: eliminate the `have` + `simpa` pattern. Instead, inline the proof
directly onto the goal after `rw [h_range_eq]`, using
`Submodule.finrank_sup_add_finrank_inf_eq` + `omega` on the goal:

```lean
-- ❌ FAILS: have h_fin : Module.finrank F (sId ⊔ kerPhi ...) = n+5 := ...; simpa
-- ✅ WORKS: rw [h_range_eq]; ... (inline proof); omega
```

Also: when `sId` is a `let` binder, goal shows it unfolded but `h_eq` from
`Submodule.finrank_sup_add_finrank_inf_eq` uses the opaque binder. Solve with
`omega` on the unfolded form, then `simpa [sId]`:

```lean
  have h_target : Module.finrank F ↥(sId ⊔ kerPhi ...) = n + 5 := by omega
  simpa [sId] using h_target
```

Discovered 2026-06-30 during the Centering.lean `finrank_halfDer` fix.
These lemma names appeared in an older Centering.lean proof that was never actually
compiled (stale olean cache).  The correct mathlib4 lemmas are:
- `Submodule.finrank_sup_add_finrank_inf_eq` (needs `[FiniteDimensional K s] [FiniteDimensional K t]`)
- `FiniteDimensional.of_finrank_pos` (to get `FiniteDimensional` from `0 < finrank`)
- `FiniteDimensional.of_finrank_eq_succ` (if `finrank = n.succ`)

**Pitfall: `split_ifs` goal direction is opposite of `IsValidSupp` lemmas.**
After `unfold coeffOf; split_ifs`, the false-branch goal is `0 = ...`,
but `IsValidSupp` lemmas return `... = 0` — the REVERSE direction.
Use `.symm` on all branches.  See: `references/split-ifs-isvalidsupp-symm.md`.

**Release audit workflow** — systematic 5-priority post-proof engineering audit.
See: `references/release-audit-workflow.md`.

**Pitfall: LSP `lean_build` passes but terminal `lake build` fails — STALE OLEAN CACHE.**
When you change a file's `open` directives (e.g. remove `open MatIdx`), the LSP's
`lean_build` tool may report success using cached `.olean` files from before the change.
But `lake build <Module>` in terminal fails because the dependencies' oleans were
compiled with the old namespace resolution.  **ALWAYS verify with `terminal(lake build)`**
after changing `open` directives or imports — do not trust LSP-only build results.
The full `lake build` (no target) forces a consistent rebuild and catches stale cache
issues.  Diagnostic pattern: LSP `lean_build` → 0 errors, terminal `lake build <Module>`
→ `error: Lean exited with code 1`, terminal `lake build` → 0 errors (rebuilds all).

**⚠️ `lake build` may silently succeed on stale cache too.** If a module was compiled
once (e.g. before `lake clean`), subsequent `lake build` reuses the old `.olean` without
recompiling, even if the source was later edited. A green build does NOT prove the
current source compiles. **To verify a specific module from scratch:** rename or delete
its `.olean`, then rebuild that module alone: `lake build E.Classification.Centering`.
The real test is `lake clean && lake build` — but this is expensive (hours for mathlib).
Prefer targeted `.olean` deletion over `lake clean`.

**CRITICAL: AGENTS.md build-status claims may be unreliable.**  When an AGENTS.md
checkpoint says "lake build = N jobs, 0 errors", this claim may have been made
against a STALE `.olean` cache — the module may never have been actually compiled
against the current source.  If you suspect this, run `lake build <Module>` (not
`lake clean` which destroys all oleans and forces a 30+ minute full mathlib rebuild).
If `lake build <Module>` fails with errors about identifiers not found or typeclass
instances missing, the checkpoint claim was based on stale cache.  The fix: diagnose
and repair WITHOUT `lake clean` first — use `lake build <Module>` iteratively until
it passes, THEN run `lake build` for consistency.

**⚠️ `lake clean` + full `lake build` is the ONLY way to definitively verify.**
As of 2026-06-29, the first clean build of the `e/` project revealed ~18 pre-existing
compile errors in Centering.lean alone, plus `Max (Type u)` issues in FinrankHelper.
All previous "0 errors" claims were stale-cache artifacts.  Use `lake build <Module>`
for targeted verification during development; reserve `lake clean` + full rebuild for
milestone audits when you have a 30-60 minute window.

**Pitfall: `dim Der(N_n) = 2n-1` is WRONG for strictly upper triangular N_n.**
The correct formula is dim Der(N_n) = n(n-1)/2 + 2n - 3 (Leger--Luks).
The 2n-1 number belongs to a different algebra. The half-derivation paper
does not depend on this formula, but introductory comparisons or lecture
notes must use the correct one. When citing "well-known" results, verify
against primary sources — don't propagate errors from memory.

**Pitfall: `lake clean` is catastrophic for mathlib projects.**  Running `lake clean`
deletes ALL `.olean` files including mathlib's ~2900 modules.  Rebuilding from scratch
takes 30–60 minutes.  NEVER run `lake clean` unless you explicitly need a full rebuild
and have the time budget.  Prefer `lake build <Module>` to test specific modules;
it only rebuilds changed dependencies.  If you suspect stale cache, `touch` the
suspicious `.lean` file and rebuild just that module.

**Pitfall: LSP auto-rebuilds mathlib after `lake clean`, competing with manual build.**
When `.lake/build` is cleaned, the running LSP server (`lake setup-file`) detects
missing `.olean` files and spawns `lean` workers to rebuild mathlib in the background
(10-15 workers, ~1GB each). These compete with any manual `lake build` for CPU/memory.
If the manual build is killed mid-flight, orphaned workers survive (reparented to PID 1)
and the LSP spawns more. **Before a clean rebuild, kill the LSP server first**, then
kill any orphaned mathlib workers, then proceed with `lake clean && lake build`.
After the build, the MCP bridge will restart the LSP as needed.
Full 5-step procedure: `references/lsp-clean-rebuild-procedure.md`.

**Pitfall: `pkill -f` can self-match.** The pattern `pkill -f '/lean.*mathlib'` may
match the `pkill` process itself, causing it to receive its own signal. Safer:
enumerate PIDs with `ps aux | grep ... | awk '{print $2}' | xargs kill`.

**Pitfall: `lake build` reports 0 errors from CACHED oleans — clean build reveals real errors.**
When `lake build` succeeds (e.g. "2967 jobs, 0 errors") but the module was NEVER actually
compiled with the current source, it's relying on stale olean files from before the new code
was added. Running `lake clean` (which deletes all `.lake/build`) and rebuilding from scratch
reveals pre-existing compile errors that the cached oleans masked. **Before claiming a
theorem is "build-verified", run a clean rebuild.** Even `lake build <Module>` can return
stale results if a dependency's olean is cached. Diagnostic: a theorem's proof uses
functions that require typeclass instances (`AddCommMonoid`, `Module`) that don't exist
in the source — the code could never have compiled, yet `lake build` passed because the
module's `.olean` was never regenerated. See `references/stale-olean-cache-pitfall.md`.

**Pitfall: `HalfDerivation` lacks `AddCommMonoid` and `Module` instances.**
The `HalfDerivation` structure in `E/Core/HalfDerivation.lean` has standalone `Zero`,
`Add`, `Neg`, `Sub`, `SMul` instances but NOT `AddCommMonoid`, `AddSemigroup`, `AddMonoid`,
or `Module`. This means `LinearMap` (via `→ₗ[F]`) cannot be constructed with `HalfDerivation`
as the domain — the `coeffOf_map` definition in Centering will fail with
`failed to synthesize instance AddCommMonoid (HalfDerivation F n)`. Fix: add
`AddCommMonoid` and `Module` instances using a `coeff_inj` helper lemma (since
`HalfDerivation` has no `@[ext]` lemma). The proofs are pointwise because `coeff` values
live in the field `F`. Also add `@[simp] lemma coeff_zero` for the `Zero` instance.
See `references/stale-olean-cache-pitfall.md` and `references/halfderivation-instance-pitfalls.md`.

**Critical: this also applies to `lake build` itself when no files changed.**
If a project reports "lake build = 2967 jobs, 0 errors" in documentation but
actually contains compilation errors (missing instances, wrong proof directions),
the entire build was served from stale `.olean` cache — no file was touched, so
no recompilation occurred, so no error was caught. The error surfaces ONLY after
`lake clean` forces a true rebuild. **Audit rule: before trusting any
AGENTS.md completion claim, either (a) touch a key file to force recompile,
or (b) `lake clean` + rebuild.** A `lake build` that completes in seconds
with "0 errors" is a red flag — it means nothing was actually compiled.

**For Chinese paper translation** (glossary-first, two-phase methodology, 10 frozen translation principles, Chinese math paper style conventions, per-section freeze discipline): see
  `references/chinese-paper-translation-workflow.md`.

**For writing mathematics for undergraduates** (three principles, 8 common student confusion points, lesson structure, common pedagogical errors to avoid): see
  `references/math-paper-pedagogy.md`.


**For Lean–Paper correspondence** — the canonical reference mapping every paper lemma to its Lean proof: see
  `references/paper-lean-consistency-audit.md`.

**For the Chinese translation of the paper** — the `paper-cn/` directory
contains a full Chinese version of the paper (§§1–8 + glossary + workflow),
translated following a strict glossary-first methodology. Load the
`chinese-math-paper-translation` skill for the translation workflow and
terminology conventions. The Chinese PDF compiles with `tectonic` via
`\usepackage{ctex}`.

**Pitfall: `split_ifs` goal direction vs `IsValidSupp` / property lemmas.**
When proving `coeffOf D i j u v = f i j u v`, the pattern `unfold coeffOf; split_ifs`
produces success cases (`rfl`) and failure cases where the goal is `0 = f i j u v`.
But lemmas like `IsValidSupp.2.1` or `k.property` return `f i j u v = 0` — the
REVERSE direction. The proof fails silently:

```lean
-- ❌ FAILS: goal is 0 = k.val i j u v, but lemma gives k.val i j u v = 0
  · exact k.property.2.1 i j u v h_invalid
-- Goal: 0 = ↑k i j u v
-- Term: ↑k i j u v = 0    ← WRONG DIRECTION

-- ✅ FIX: use .symm
  · exact (k.property.2.1 i j u v h_invalid).symm
```

This is especially deceptive because (a) it compiles in some Lean versions
depending on `split_ifs` behavior, and (b) it appears in AGENTS.md checkpoints
that claim "compiles" but actually relied on stale olean caches.
Always verify `split_ifs` goal directions after `unfold`.
`obtain ⟨huc, hvd⟩ := h1` where `h1 : P ∧ Q` removes `h1` from the context.
Later `simpa [h1, ...]` (needed to simplify `if`-conditions in bracketLeft etc.)
fails with "Unknown identifier h1".  Do NOT `obtain`.  Use `.left` / `.right`
for component access and keep `h1` intact:
```lean
  by_cases h1 : u < c ∧ v = d
  · have hpos := coeffOf_f ... h1.left (by omega)
    simpa [h1, hpos]
```
Also: when a hypothesis has a lambda-wrapped form like
`hne : (fun i j u v => coeffOf D i j u v) i j u v ≠ 0`,
`rw [hcoeff] at hne` fails to find the pattern.  Use
`have hne' := by simpa [hcoeff] using hne` instead.
See `references/coeffOf-bridge-pattern.md` for the full pattern.

**For proof extraction from computational data** — when a coefficient resists all local proof attempts
but computational nullspace analysis confirms it must be zero, use the syzygy extraction methodology
to find the minimal set of half-Leibniz equations that force the result. See
  `references/computational-experimentation.md`.

**For the Boundary Rigidity proof** — the 3-equation syzygy that closes the v=n case without
I_filtered, kerΦ, or width_b_zero. See
  `references/boundary-rigidity-proof.md`.

**For the adjacent-source v=n five-class dispatch** — the complete classification and proof of all five position classes for adjacent source (i,i+1) at target column n: u=i (BoundaryRigidity), u=i+1 (single equation), u>i+1 (two-equation syzygy, char≠3), u<i (single equation), i=n-1/Pattern D (reduces to v<n). All proven from Phi=0 alone without I_filtered/kerΦ/centering. See
  `references/adjacent-vn-five-class-dispatch.md`.

**For the nonadjacent-source v=n zero pattern** — 4-case dispatch (coeffOf_source_exists) proving `coeffOf D i j u n = 0` for j-i≥2, Phi-only, no induction needed for IB (adjacent_above_zero closes child). See
  `references/nonadjacent-vn-zero-pattern.md`.

**For the Sorrie 3 2-equation syzygy** — closes adjacent-source u>i at v=n via two half-Leibniz equations with the same partner (i,i+1): source=(i,u) gives coeff=coeff, source=(i+1,u) gives coeff=−2·coeff, yielding 3·coeff=0. Char≠3 dependent (distinct from BoundaryRigidity's char≠2). Uses T3 mechanism where coeff(i,i+1;u,n) appears as T3 when source has column=u. See
  `references/sorrie3-two-equation-syzygy.md`.

**Assembly-first strategy — skeleton before drilling** (when a proof node has multiple sorries, write the COMPLETE assembly skeleton first, wiring all existing lemmas into the final theorem statement.  The skeleton reveals interface gaps (e.g. missing `IsValidSupp` bridges) that are invisible when drilling into individual sorries.  Only after the full dependency graph is concrete should you decide which sorries are truly blocking.  This was the key strategic move in D3 (2026-06-28): writing `centered_part_mem_kerPhi` and `halfDer_decomposition` skeletons before attacking `v=n`.)

**For the first-row family structural audit methodology** — when a coefficient
resists all standard approaches (source-shift, partner audit, local SCC), switch
to family-level enumeration: fix the target row, map ALL T1-T4 couplings, prove
structural invisibility, and find multi-step chains via coeffOf_cond + IH.
See `references/first-row-family-audit.md`.

**Pitfall: i=1 source-shift degeneracy.** When applying the T3 source-shift with
source (1, i+1) and partner (i, i+1), the case i=1 makes them the SAME source.
T2 and T3 both fire, giving -goal + goal = 0 (tautology). Split i=1 before the
source-shift: i=1,u=i+1 uses h_notI contradiction ((2,n)∈I); i=1,u=i+2 uses
boundary_coupling directly. See `references/t3-cross-source-elimination.md`.

**For the 2×2 SCC via coeffOf_cond C-term** — when bracketIdentity gives only ONE
independent equation (all partners produce the same relation), the second equation comes
from coeffOf_cond whose C-term ALWAYS fires when target_row = source_row and split_k <
target_col.  Forms a 2×2 system {x=y, 2x=y} with det=1, char-independent.  Discovered
during boundary_family_right u≥4 audit (2026-06-27).  See:
  `references/scc-coeffOf-cond-pattern.md`.\n\n**For the BND-L 3-equation local SCC** — purely local 3×3 system
(bracketIdentity×2 + coeffOf_cond×1), no hI/induction/width, det=1
char-independent, uniform family reduction via coeffOf_cond at k=4, and the
commuting-partner bracketIdentity technique for isolating goal coefficients:
  see `references/bnd-l-3equation-scc.md`.

**Engineering discipline (replace-then-verify):** when rewriting a lemma with
new infrastructure, keep the OLD proof archived (e.g. `_old` suffix). Replace
only after the new proof compiles and all callers are verified. Verify after
EVERY step with `lean_diagnostic_messages(severity='error')`. Delete old proof
only after full dispatch closure.

**For the internal-det-coupling 3×3 SCC** (adjacent-source u≥3 interior coefficient closure via 2 bracketIdentity + 1 coeffOf_cond, det=-1, char-independent — and the systematic partner-audit methodology that discovered it): see
  `references/internal-det-coupling-3x3-scc.md`.

**For the wider C-fire dependency probe** — tracing the full coefficient chain
from coeffOf_cond children through diagonal intermediates to b-channel to parent,
determining whether a child is locally closeable or creates a structural cycle,
and the key finding that width-2 is special (no intermediate diagonal) while
wider sources introduce cycles: see
  `references/c-fire-wider-chain-analysis.md`.

**For boundary dispatch analysis** — BND-L local 3×3 SCC (boundary_local_234),
BND-R dispatch unreachability (architectural dead branch), bracketIdentity
partner-selection technique and "free coefficient" detection (when all T1-T4
are trivially zero, the constraint comes from bracketSource coupling to
nonadjacent coefficients): see
  `references/boundary-dispatch-analysis.md`.

**For BND-L boundary dispatch hI-free closure** — the 3-equation SCC that
replaces the blueprint's hI-dependent `width_stability_c` call for wide ∈I
residuals. Self-contained system (coeffOf_cond + 2 bracketIdentities),
**For post-proof mathematical audit methodology** (foundational specification, object-level auditing, phased structure Phase 0→A→F, dim=n as canonical example of object-layer confusion, coordinates-are-not-objects rule, image/kernel > span/condition): see
  `references/foundational-specification-methodology.md`.

**For paper writing rigor audit** — when converting Phase design documents
into a publishable paper, audit each section for: undefined objects, theorem
dependency closure, "obviously/clearly" claims without justification, and
incomplete proofs. The §2–§7 audit of the HalfDer paper found 18 issues
across 6 sections, including notation errors, circular dimension counts,
sketch proofs, and Phase-document references in the paper. See:
  `references/paper-rigor-audit.md`.

**Pitfall: conjectured formula ≠ Lean mechanism.** When a conjectured
formula (e.g. `φ_i = 2·φ_{i+1}`) resists proof, audit the Lean code first.
The Lean mechanism may be fundamentally different (e.g. `coeffOf_cond`
split-k recursion vs. simple factor-2 jump). Do NOT try to prove the
conjectured formula — establish correspondence between Lean and Paper
instead. See: `references/lean-paper-correspondence.md`.

**For paper writing rigor audit** — when converting Phase design documents
into a publishable paper, audit each section for: undefined objects, theorem
dependency closure, "obviously/clearly" claims without justification, and
incomplete proofs. Common issues: notation errors, circular dimension counts,
Phase references in paper, missing formal definitions. See:
`references/paper-rigor-audit.md`.

**For mathlib API migration** — renamed/removed lemmas, import path changes,
and structure instance pitfalls discovered during the v4.31.0-rc1 migration: see
  `references/mathlib-api-migration.md`.

**For import-isolation pattern** — when a lemma compiles in isolation but
fails in a file with many imports due to universe/typeclass conflicts
(e.g., `Max Type` errors on `Submodule F (ℕ⁴ → F)`): see
  `references/import-isolation-for-typeclass-conflicts.md`.

**For mathlib finrank sup compatibility** — the working pattern for
`Submodule.finrank_sup_add_finrank_inf_eq` with `LinearEquiv.finiteDimensional`
to provide `FiniteDimensional` instances.  Updated 2026-06-30 with the
`LinearEquiv.finiteDimensional` direction pitfall: see
  `references/mathlib-finrank-sup-compatibility.md`.

**For Chinese paper translation and formatting** — methodology, terminology, two-phase workflow, and thesis-style formatting with ctex: see
  `references/chinese-paper-translation.md`.

**For paper writing for undergraduates** — three-version strategy, layered navigation, zero-prerequisite principle, 8 common student confusion points and their resolutions, and the "define everything from scratch" methodology: see
  `references/math-paper-pedagogy.md`.

**For the BND-L 3×3 local SCC mechanism** (boundary local closure, hI-free, det=1, char-independent; three commuting-partner bracketIdentity equations + one coeffOf_cond): see
  `references/bndl-3eq-scc-closure.md`.

**For Chinese paper translation** (glossary-first, two-phase methodology, 10 frozen translation principles, Chinese math paper style conventions, terminology table): see
  `references/chinese-paper-translation.md`.
The `paper-cn/` directory holds the Chinese translation artifacts alongside the English `paper/`.

---

## 🔷 Foundational Specification Methodology

When the user asks to re-audit, restructure, or formalize the mathematical
foundations of a Lean project (as opposed to filling individual sorries), use
the following methodology. This emerged from the v1.0-proof-complete re-audit
that identified the dim=n error as a failure of the constraint space (SEq)
rather than a missing proof case.

### Core Principle: Object Separation

Every mathematical entity belongs to exactly one category:
- algebraic object,
- linear map,
- image,
- kernel,
- coordinate representation,
- logical statement.

Transformations between categories are always realized by explicitly defined
morphisms. No later phase is allowed to identify entities from different
categories without exhibiting such a morphism.

**Coordinates are not objects.** Names like A_i, B, C, D, E, F are coordinate
labels introduced in the final phase only. Earlier phases construct basis
elements e_α (linear maps N→N) and refer to them by index.

### Phase Discipline

Organize the audit/restructuring into sequential, non-overlapping phases:

```
Phase 0: Object Diagram — all spaces and morphisms
Phase A: Solution Space — explicit basis construction
Phase B: Constraint Space — generating set enumeration
Phase C: Morphology — decomposition of each generator
Phase D: Syzygy — linear relations among generators
Phase E: Minimal Basis — selection of independent generators
Phase F: Coordinates — incidence matrix, classification
```

Each phase:
- Depends only on phases above it (DAG, no cycles).
- Has an explicit Scope section listing what it establishes and what it does NOT.
- Has an Exported Results section listing deliverables to the next phase.
- Ends with "canonical decomposition deferred to Phase X" for anything beyond scope.

### Blueprint → Audit → Freeze Pattern

**Never write a formal document first.** The progression is:

1. **Blueprint**: Answer foundational questions with no case analysis, no
   coordinate names. Define what objects are, what arrows exist, which spaces
   are images/kernels. Use `Im(·)` and `ker(·)` rather than `span{...}` for
   structural objects.
2. **Audit**: Check object closure (all names trace to definitions), arrow
   closure (all maps have domain/codomain), dependency closure (DAG, no
   reverse edges), uniqueness (no two names for the same object).
3. **Freeze**: After audit passes, stamp the document as FROZEN with a version
   and date. No further modifications permitted. All subsequent phases
   reference it immutably.

### The dim=n Error Pattern

The dim=n error was NOT a missing case in a proof. It was:

```
SEq_{dim=n} = Im(π*∘Ψ♯|_{restricted domain})  ⊊  Im(π*∘Ψ♯) = SEq
```

An entire class of constraints (non-local commuting pairs like (E_1, E_{n-1}))
was absent from the constraint space because the domain was improperly
restricted. The error was at the **Equation Universe** level, not the proof
level. When re-auditing, always check: is the constraint space fully generated,
or was the domain artificially restricted?

### User preferences for this class of work

- Discussion in Chinese, Lean code and mathematical notation in English.
- Prefer `Im(·)` and `ker(·)` over `span{...}` for structural definitions.
- Use curried forms (Ψ♯, Ψ♭) and pullbacks (π*) rather than ad-hoc notation.
- No case analysis before the object diagram is frozen.
- No coordinate names (A_i, B, C, ...) before the final phase.
- No "force," "therefore," "implies" language in descriptive phases (C and earlier).
- Add `## Exported Results` and `### NOT exported` tables at every phase boundary.
- The user strongly prefers precise, qualified claims over overstated ones.

Full methodology reference: `references/foundational-specification-methodology.md`.

**Pitfall: `dim Der(N_n) ≠ 2n-1`.** For strictly upper triangular $N_n$,
the correct formula is $\dim\Der(N_n)=n(n-1)/2+2n-3$ (Leger--Luks).
The $2n-1$ result is for a different algebra. Our HalfDer proof never
uses $\Der$, but citing the wrong number in background/lecture notes
misleads readers. Check: for $n=5$, $\dim N_5=10$, $\dim\Der=17$ (not 9).

**For σ-mirror audit methodology** (N_n symmetry: when mirroring a proof, check coeffOf_cond survivor remapping, commuting-partner recomputation, branch disappearance, and lemma delegation — do NOT mechanically copy): see
  `references/sigma-mirror-audit.md`.

**For `internal_det_coupling_zero` descending-chain mechanism analysis** — systematic bracketIdentity enumeration showing why the naive adjacent-source chain fails (sources share index → don't commute), and the alternative via partner `(i,n)` producing `coeff(u-1,u; u-1,i) = coeff(i,n; u,n)`: see
  `references/internal-det-coupling-chain-analysis.md`.

**For dependency audit workflow** (when sorries drop to ~15, map all remaining into core-SCC vs assembly vs deferred tiers before attacking any single one): see
  `references/dependency-audit-workflow.md`.

**For blocker-verification audit** (when stuck on exactly 1 LIVE sorrie that resists multiple approaches — verify it actually blocks real downstream theorems, not just placeholders, and cross-reference the paper's proof structure before attacking): see
  `references/blocker-verification-audit.md`.

**For assembly audit** (when local lemmas are all proven and a skeleton proof body exists — verify that the dispatch tree covers every leaf, no coverage gaps, and no interface mismatches, BEFORE claiming "math is complete"): see
  `references/assembly-audit-methodology.md`.

**Pitfall: over-stating completion status.** Do NOT say "mathematics complete, only Lean wiring remains" unless you have performed an assembly audit confirming every leaf of the target theorem's dispatch tree maps to an existing theorem with compatible interface. Prefer precise formulations: "local lemmas proven (0 sorries); assembly audit shows dispatch tree is complete; N tactical errors remain."

**For proof-state audit before attacking a sorrie** (three-step methodology: interface check → proof-state trace → gap map / freedom DAG; watch for the "theorem too strong" anti-pattern where a monolithic case split obscures that most sub-cases are already handled): see
  `references/proof-state-audit-methodology.md`.

**For experimental verification before proof search** (when a MATH-classified blocker needs truth-value disambiguation — build the full constraint matrix, compute nullspace dimension, compare with known answer to determine World A vs World B): see
  `references/experimental-nullspace-verification.md`.
  Reusable script: `scripts/compute_nullspace_dim.py`.

**For proof extraction from constraint matrix RREF** (when experimental nullspace confirms World A but no local equation constrains the target — extract the exact linear combination of equations = a syzygy — that forces the coefficient to zero). Reusable script: `scripts/extract_syzygy.py`. Full methodology: see
  `references/proof-extraction-from-constraint-rref.md`.

> **Design rule (2026-06-26):** σ preserves the internal_det structure (clean mirror:
> `internal_det_step` → `internal_det_step_a`) but does NOT preserve the width-chain
> case decomposition (`width_c` → `width_a`: B disappears, C shifts k=2→k=1,
> D appears at k=n-2).  Always re-audit survivors after σ.

**For consistency-auditing the domain of a theorem** — when the blueprint/paper
describes a different domain (e.g. centered component) than the Lean statement
(e.g. arbitrary D), audit before filling sorries. The Id counterexample pattern:
check whether Id violates the theorem by computing coeffOf(Id, i, j, i, j).
If yes, the Lean statement needs a domain qualifier (h_off_diag, h_centered, etc.)
that matches the blueprint. See `references/consistency-audit-before-engineering.md`.

**Expansion-first methodology — compute before conjecturing.**
When designing a proof strategy for a propagation or syzygy claim, do NOT
start by conjecturing a formula like `φ_i = 2·φ_{i+1}`. Instead, pick a
concrete small parameter set (e.g. i=1,j=3,l=4), expand Ψ fully into
coefficient form, and check whether the formula even holds. Raw expansion
often reveals that the conjectured form is wrong while the Lean code's
mechanism (e.g. `coeffOf_cond` split-k recursion) was correct all along.
See: `references/lean-paper-correspondence.md`.

**Pitfall: three status labels only.** Mathematical claims must use exactly
one of: Established, Conjectured, Conditional. Never "Key finding," "Key
observation," or percentage-based progress. Report counts, not percentages.

**Proof status reporting convention.** Never report proof progress as a
percentage (e.g. "~70%"). Instead, report exact counts:

```
Established:  N
Core Gaps:    M
Sketch:       K
```

Percentages mislead because one lemma may be nearly complete while another
hides an undiscovered structural obstacle. Counts are objective.

**Expansion-first methodology — compute before conjecturing.**
When designing a proof strategy for a propagation or syzygy claim, do NOT
start by conjecturing a formula like `φ_i = 2·φ_{i+1}`. Instead, pick a
concrete small parameter set (e.g. i=1,j=3,l=4), expand Ψ fully into
coefficient form, and check whether the formula even holds. Raw expansion
often reveals that the conjectured form is wrong while the Lean code's
mechanism (e.g. `coeffOf_cond` split-k recursion) was correct all along.
See: `references/lean-paper-correspondence.md`.

---

## 🔷 Lean–Paper Correspondence

When a conjectured formula resists proof and the Lean codebase implements
a different mechanism, do NOT try to prove the conjectured formula. Instead,
establish a **correspondence** between the Lean implementation and the
paper's mathematical language.

### The Pattern

1. **Audit the Lean code.** Read the key lemma (e.g. `coeffOf_cond`) and
   trace it back to the half-Leibniz axiom it invokes.
2. **Compute a concrete expansion.** Pick a specific parameter set, expand
   Ψ fully into coefficient form, and verify term-by-term that the Lean
   lemma IS the coefficient expansion of a specific Ψ equation.
3. **State the correspondence.** E.g. `coeffOf_cond = Ψ(E_ik, E_kj, T) = 0`.
   This proves that the Lean mechanism and the paper's equation are the
   SAME object in different languages.
4. **Inherit Lean results.** Once correspondence is established, the paper
   inherits the Lean proof. The paper does not re-prove what Lean already
   proved — it proves equivalence, then cites Lean.

### Three convergence paths for proof obligations

Every conjectured syzygy resolves via one of:
- **Proved directly** (explicit expansion/computation).
- **Established via correspondence** (Lean lemma = paper equation).
- **Vacuous after structural analysis** (conjecture assumed N non-zero
  generators, only 1 exists).

### Key example

The conjectured `φ_i = 2·φ_{i+1}` (simple BL→BL propagation) was disproven
by explicit Ψ-expansion. The Lean code's `coeffOf_cond` (split-k recursion)
was the correct mechanism. The correspondence `coeffOf_cond = Ψ(E_ik, E_kj, T) = 0`
unified the paper's three-step chain with Lean's recursion.

---

## 🔷 Six-Layer Project Philosophy

Every complex proof project decomposes into layers. Each layer answers
exactly one mathematical question. Layers must not be mixed.

| Layer | Question | Typical Work |
|-------|----------|-------------|
| Result | Why is the answer not what we thought? | Identify the nature of the error |
| Object | What are we studying? | Phase 0: object diagram, spaces, morphisms |
| Language | Why do Paper and Lean look different? | Correspondence audit |
| Proof | What actually has to be proved? | Isolate obligations into a DAG |
| Presentation | Why should the reader find this inevitable? | Narrative organization, readability audit |
| Method | How should we begin next time? | Reusable workflow |

### Execution Rule (after architecture freeze)

Once Phases 0–F are frozen:
- No new mathematical objects.
- No new architectural phases.
- No modification of 0–F unless a logical contradiction is discovered.
- All new work is either (1) proving a deferred lemma, or (2) assembling
  previously proved results.

### Conjecture Dependency Rule

Any Phase E/F/G reference to a Conjecture (Phase D §D.3.x) MUST be
explicitly marked "Assuming Conjecture D.3.x." Conjectures may not be
cited as established facts.

**Pitfall: `cases` on a function-equality hypothesis (not `rw`).**
resists proof and Lean implements a different mechanism, map the Lean
lemma to its Ψ-equation equivalent. Three convergence paths: (1) proved
directly, (2) established via correspondence, (3) vacuous after structural
analysis. The `coeffOf_cond = Ψ(E_ik, E_kj, T) = 0` identity is the
canonical example. See: `references/lean-paper-correspondence.md`.

**Pitfall: "candidate" vs "established" — never claim a theorem when it's conjectured.** When a structural claim has not been proved, label it "Conjecture" or "Candidate." Do not write "Key finding" or "Key observation" for unproven claims. Do not use percentages to describe proof progress — report counts: Established / Core Gaps / Sketch. Only three status labels are allowed for mathematical claims: Established, Conjectured, Conditional.

**Pitfall: audit before freeze.** Every Phase must undergo an audit (Responsibility Audit, Theorem Audit, or Object Purity Audit) before receiving the FROZEN stamp. The audit checks for: phase boundary leakage, premature conclusions, new object creation, and interface contract violations. Fix ALL audit findings before freezing.

**For the computational-experiment → syzygy-extraction → parameterized-proof methodology**
that discovered Boundary Rigidity: see `references/boundary-rigidity-proof.md#discovery-methodology`.
Key pattern: (1) build full linear system, (2) compute nullspace to verify truth,
(3) extract minimal syzygy via RREF pivot tracing, (4) generalize to parameterized Lean theorem.

**Pitfall: `split_ifs <;> omega` works in file compilation but fails in `lean_run_code`.** When proving bracketIdentity/bracketSource expansions with concrete numeric indices in `lean_run_code`, the pattern `split_ifs <;> first | omega | ring | simp` (used throughout BoundaryRigidity.lean) often fails because `omega` cannot resolve numeric inequalities from the `split_ifs` hypothesis context in isolated code snippets. The `split_ifs` generates contradictory hypothesis combinations that `omega` doesn't see as contradictory. **Remedy**: use `simp` with explicit `by omega` hypotheses instead:
```lean
-- FAILS in lean_run_code:
example : bracketIdentity coeff 3 6 3 4 3 8 = ... := by
  rw [bracketIdentity_eq_expanded]
  split_ifs <;> first | omega | ring | simp  -- omega can't close all branches

-- WORKS everywhere:
example : bracketIdentity coeff 3 6 3 4 3 8 = ... := by
  rw [bracketIdentity_eq_expanded]
  have h4 : (4 : ℕ) < 8 := by omega
  have h5 : (6 : ℕ) < 8 := by omega
  have h_not4 : (8 : ℕ) ≠ 4 := by omega
  simp [h4, h5, h_not4]  -- clean, omega resolves the hypotheses first
```
The `simp` with explicit hypotheses approach works in both `lean_run_code` and file compilation, while `split_ifs <;> omega` is context-sensitive. Prefer the explicit approach for verification snippets. `linarith` supports ℕ, ℤ, ℚ, ℝ only.
For general Field F, use `eq_of_sub_eq_zero` + `ring` + `calc` for algebraic manipulations.
From `2*a - b = 0`, use `(eq_of_sub_eq_zero h).symm` to get `b = 2*a`.
For `ring_nf`: it may reorder factors (e.g., `4*a` → `a*4`); use `mul_comm` to adjust.
proofs in the adjacent-source dispatch, step back and compare the Lean
implementation's induction structure against the paper's. The paper proves
image containment via SOURCE-WIDTH induction (CE-generation lemma) which
has never been implemented in Lean. The skeleton uses a different induction
(target-width) and a per-target case split (u>i/u=i/u<i) that creates
artificial difficulties (e.g. L2099 first-row rectangular coefficients).
See `references/ce-generation-vs-skeleton-divergence.md`.

**For proof design audit** — when per-coefficient proof search stalls, use the
"DAG reconstruction" technique: (1) draw the paper's propagation DAG, (2) map
every Lean lemma to an edge, (3) identify missing/gap edges vs. case-split
artifacts. This revealed that ImageContainment.lean is an ORPHAN module (zero
importers) whose skeleton diverges from the paper's structure. See
`references/ce-generation-vs-skeleton-divergence.md`.

## 🔴 Workflow Discipline (session-corrected rules)

- **INVESTIGATE BEFORE MODIFYING — evidence-first debugging.** When facing
  multiple complex Lean errors (typeclass / Max Type / elaboration failures),
  do NOT start patching code. First collect FULL evidence: (1) read source
  context ±30 lines around each error, (2) get LSP goals at each error site,
  (3) verify mathlib API existence with `#check`/`#print` (many lemma names
  in old proofs were phantom), (4) diff against pre-session backups to
  classify errors as pre-existing vs newly-introduced, (5) build a diagnostic
  matrix: confirmed / plausible hypothesis / cascade. Only after the matrix
  is complete should any code be edited. See `references/d4-centering-debug-case-study.md`.

- **CLAIM DISCIPLINE — hypothesis vs confirmed.** When a diagnostic says an
  error is a "cascade" of another, it is a HYPOTHESIS until a rebuild after
  fixing the upstream error proves the downstream error disappears. Use
  precise language: "root cause hypothesis" not "100% confirmed". The user
  will call out premature certainty — always qualify claims before rebuild
  verification.

- **IMPORT GRAPH ≠ PROOF GRAPH.** Module A importing Module B does NOT mean
  the main theorem depends on B's proofs. Only theorems actually called in
  proof bodies matter. Trace the theorem dependency DAG (expensive,
  theorem-level) after establishing the import closure (fast, module-level).
  Sorries in imported-but-never-called modules are NOT blocking.

- **BEFORE DELETING "DEAD" MODULES: verify 0 importers.** Call-graph showing
  0 callers ≠ safe to delete. Modules may be imported for type definitions.
  Run `search_files(pattern='import.*ModuleName', target='content')` across
  the entire project. In one audit, 11/24 candidates were imported by live code.

- **`Module.finrank` on Submodule sup in `have` types triggers elaboration failure.**
  When `Module.finrank F (sId ⊔ kerPhi ...)` appears as the TYPE of a
  `have` statement, Lean can fail with "Max Type" even though the same
  expression works as a goal. Fix: eliminate the `have` + `simpa` wrapper;
  inline the proof directly onto the goal. See `references/d4-centering-debug-case-study.md`.

- **INVESTIGATE BEFORE MODIFYING — evidence-first debugging.** When facing
  multiple complex Lean errors (especially typeclass / Max Type / elaboration
  failures), do NOT start patching code. First collect FULL evidence: (1) read
  source context ±30 lines around each error, (2) get LSP goals at each error
  site, (3) check mathlib API existence (`#check`/`#print` — many "known" lemma
  names are phantom), (4) diff against pre-session backups to classify errors as
  pre-existing vs newly-introduced.  Only after producing a **diagnostic matrix**
  with confirmed/plausible/cascade classifications should any code be edited.
  This session proved: what looked like a "Max Type universe bug" was actually a
  `LinearEquiv.finiteDimensional` direction error + `have`-type elaboration issue.
  Both would have been invisible without systematic investigation.

- **CLAIM DISCIPLINE — hypothesis vs confirmed.** When a diagnostic says an error
  is a "cascade" of another, it is a HYPOTHESIS until a build after fixing the
  upstream error proves the downstream error disappears.  Use precise language:
  "root cause hypothesis" not "100% confirmed".  Only after `lake build` succeeds
  can you say "root cause confirmed."  The user will explicitly call out premature
  certainty — always qualify claims that haven't been verified by a rebuild.

- **IMPORT GRAPH ≠ PROOF GRAPH.**  Module A importing Module B does NOT mean
  the main theorem depends on B's proofs.  Only theorems actually called in
  proof bodies matter.  When auditing a project: (a) first establish the import
  transitive closure (fast, module-level), (b) then trace the actual theorem
  dependency DAG through proof bodies (expensive, theorem-level).  The import
  closure is a superset — often much larger than the proof dependency closure.
  Sorries in imported-but-never-called modules are NOT blocking.  This distinction
  is critical for project health assessment: the 2026-06-30 audit found that of
  19 sorries in "critical-path modules", 0 were on the actual proof graph.

- **CLEAR LSP ERRORS BEFORE FILLING SORRIES.** A file with M errors + N sorries
  blocks LSP feedback; every edit is a blind gamble. First achieve 0 errors (even
  if that means converting some errors into sorries), then fill sorries on a
  stable surface. See `references/lsp-error-fix-patterns.md` for common error
  classes and their fix recipes.

- **`rfl` / `simp` RELIABILITY DEPENDS ON BINDER MODE.**  `variable {F n}`
  (implicit) breaks definitional reduction of structure-instance field projections
  that work fine with `variable (F n)` (explicit).  `lean_run_code` is NOT a
  faithful replica of file compilation.  The fix: add `@[simp]` bridge lemmas in
  the instance-definition file (where binders are explicit, `rfl` works).
  See `references/definitional-equality-implicit-binders.md`.

- **CLASSIFY SORRIES BY DOMAIN BEFORE DEPTH-FIRST.** Off-diagonal (i≠u∨j≠v) sorries belong to the primitive layer; diagonal ones (i=u∧j=v) must wait for the centered wrapper (D3.4). Do NOT force-close diagonal sorries in the primitive layer — this creates structural cycles.

- **DO A DEPENDENCY AUDIT BEFORE CHANGING A THEOREM'S API.** Before adding hypotheses to a signature: verify callers, check blueprint/paper domain, confirm callers satisfy the proposed hypothesis. Zero-caller theorems are safest to modify.

- **DO A DEPENDENCY PROBE BEFORE DESIGNING CLOSURE MECHANISMS.** When a coefficient resists closure, enumerate its full coeffOf_cond and bracketIdentity graph BEFORE inventing new induction predicates or SCC lemmas. The probe reveals whether it's locally closeable, needs centering, or cycles back to parent.

- **PREFER LOCAL ALGEBRAIC CLOSURE OVER NEW LEMMAS.** When a residual reduces via coeffOf_cond to a single adjacent child, check for self-contained SCC with bracketIdentity. The 3-equation pattern (coeffOf_cond + 2 commuting bracketIdentities) closes hI-free.

- **STOP DEPTH-FIRST ENGINEERING WHEN A STRUCTURAL CYCLE IS DETECTED.** If filling a sorry leads to A→B→C→A, pause. The issue is architectural, not tactical. Do not add more machinery.

- **AUDIT DEPENDENCIES BEFORE DEPTH-FIRST PROOF SEARCH.** When facing multiple sorries, first check which lemmas have zero callers (`search_files` across the project). Orphaned lemmas' sorries are not live blockers — annotate them as CURRENTLY UNUSED and move on. Then classify remaining sorries as live / deferred-to-later-phase and build a minimal dependency graph. See `references/image-containment-local-scc-pattern.md`.

- **THE BLUEPRINT IS THE DESIGN SPEC.** When Lean contradicts blueprint (e.g. missing off-diagonal qualifier), blueprint wins. Fix Lean to match.

- **AUDIT ALL REMAINING SORRIES BEFORE DIVING INTO ANY ONE.** When the sorry count drops to ~15 or fewer, do a dependency audit: list every sorry with its enclosing lemma, map which call which, classify into (a) core SCCs needing new proofs, (b) assembly/wiring waiting for core lemmas, (c) legitimate deferrals to later sections (§D centering, all_diag_equal). This prevents wasted time on sorries that depend on unresolved upstream nodes.  See `references/dependency-audit-workflow.md`.

- **RECOGNIZE THE ASSEMBLY PHASE — STOP PARTNER AUDITS WHEN LOCAL SCCs ARE DONE.**
  The project transitions through distinct phases: (1) mechanism discovery (partner
  audits, local SCC invention), (2) dispatch assembly (wiring proven lemmas into
  the skeleton), (3) centering/deferred (§D). When the local SCC inventory is
  complete (8 proven mechanisms for this project), a sorrie that resists multiple
  rounds of partner enumeration is NOT a signal to keep searching — it's a signal
  that the sorrie belongs to a later phase. Before launching another partner audit,
  ask: "Is this sorrie in a dispatch slot that simply hasn't been wired yet?" and
  "Does GOAL appear in ANY bracketIdentity for ANY valid partner?" If the answer
  to the second is NO, the sorrie is almost certainly deferred to §D centering,
  not a new local SCC candidate. See `references/transfer-dispatch-mechanism-analysis.md`
  for the detailed proof that GOAL is structurally invisible for the TRANSFER case.

- **INTERFACE-CHECK BEFORE WIRING — NEVER CLAIM "LEMMA X CLOSES SORRIE Y" WITHOUT VERIFYING THE SIGNATURE.** When a new theorem is completed and designated to close a sorrie, do NOT trust the summary. Before editing any file, load the theorem signature (lean_file_outline or #check) and the goal at the sorrie site (lean_goal), then compare on three dimensions: (1) **Hypothesis compatibility** — does the theorem require preconditions the sorrie context doesn't provide? (2) **Object compatibility** — does the theorem operate on `D.coeff` while the goal needs `coeffOf`, or vice versa? (3) **Conclusion compatibility** — does the theorem's conclusion exactly match the goal shape, or is there a quantifier/index mismatch? Only after all three match (or you've identified the exact bridge needed) should you edit the sorrie file. See `references/interface-verification-before-wiring.md` for the full methodology and the BoundaryRigidity↔Centering case study where this caught a 3-layer mismatch that a summary-based claim missed.

- **ASSEMBLE THE FULL PROOF SKELETON BEFORE FILLING REMAINING SORRIES.**
  When mechanism discovery is done and the task shifts to D3/D4 assembly,
  write the COMPLETE theorem skeleton first — all sub-lemmas, all dependency
  edges, all sorries explicitly marked. Compile it. Only then analyze which
  gaps are truly live. The skeleton often exposes invisible interface gaps
  (e.g. `IsValidSupp` which no one discussed during the local-SCC phase)
  and may reveal that the strongest lemma is not actually needed. See the
  three-outcome classification (A/B/C) and the `ext_coeff` pattern in
  `references/d3-assembly-methodology.md`.

- **DON'T CLEANUP IMMEDIATELY AFTER PROOF COMPLETION.** When `lake build` passes
  with 0 errors, the project is in a stable state. Do NOT immediately delete
  orphaned theorems, refactor files, rename namespaces, or restructure directories.
  First complete a systematic release audit (see `references/release-audit-methodology.md`),
  then archive legacy code in a unified pass. Premature cleanup risks breaking
  the stable build.

**For release audit methodology** — the systematic 5-priority post-proof audit
that confirms engineering soundness before declaring a Lean formalization complete:
see `references/release-audit-methodology.md`.

**For project architecture audit** — whole-project health assessment (call graph,
dead code, tactic profile, API risk) after milestones or before refactors:
see `references/project-architecture-audit.md`.

- **"GOAL INVISIBLE" → DEFER TO §D, NOT NEW PARTNER AUDIT.**
  When GOAL = coeff(i,i+1; u,v) does not appear in bracketIdentity for any valid
  partner (proved: T1 needs c=v∧d=v→degenerate, T2 needs c=u∧d=u→degenerate,
  T3/T4 are different sources), the constraint chain goes through bracketSource →
  nonadjacent coefficient → diagonal of wide source → centering. This is a
  STRUCTURAL fact, not a missing lemma. The sorrie should be classified as
  deferred-to-§D, alongside the `u=i` interior case. Mark it clearly and move on.
  Do NOT iterate partner choices — the iteration space has been exhausted.

- **WHEN STANDARD APPROACHES FAIL, SWITCH TO STRUCTURAL FAMILY AUDIT.**
  If source-shift, partner audit, local SCC, and coeffOf_cond single-step all fail,
  and you've been iterating partners for >3 rounds, STOP. Switch to a full
  **family-level structural audit** (see `references/first-row-family-audit.md`):
  fix the target row (or other structural invariant), enumerate ALL half-Leibniz
  equations involving that family, classify T1-T4 couplings, and prove structural
  invisibility if warranted. The answer is almost certainly a multi-step chain,
  not one more partner.

- **WHEN ALL LOCAL METHODS ARE EXHAUSTED, VERIFY TRUTH BEFORE PROVING.**
  If a sorrie survives occurrence audit, partner audit, reverse dependency audit,
  and interface audit, and the blocker-verification audit classifies it as MATH:
  do NOT design a proof yet. First run experimental nullspace verification
  (`scripts/compute_nullspace_dim.py`) to determine whether the statement is
  actually true. If the dimension matches (World A), the global constraint exists
  and must be extracted. If it doesn't match (World B), the theorem statement
  needs revision. Do NOT invest in a proof before World A/B is resolved.

- **DO AN OCCURRENCE AUDIT BEFORE LOCAL PROOF SEARCH.**
  When a coefficient resists closure after 2-3 attempts, do NOT iterate partners.
  Instead, run a computational enumeration of ALL `(source, partner, target, T1/T2/T3/T4)`
  quadruples to map where the goal coefficient appears in bracketIdentity equations.
  Classify into: commuting (bracket_zero available), non-commuting (bracketSource path),
  or structurally invisible (→ defer to §D).  L2099 proved that local search routinely
  misses occurrences because the goal's column in the equation may differ from the goal's
  own column.  See `references/occurrence-audit-methodology.md` for the Python script,
  interpretation guide, and the L2099/L2136 case studies.

- **BEFORE ATTACKING A STUBBORN SORRIE, DO A MODULE-LEVEL DAG AUDIT.**
  Search whether the enclosing lemma/module actually has any downstream consumers.
  Use `search_files(pattern='<lemma_name>', target='content')` across the entire
  project. If zero callers: the sorrie is in a dead branch — annotate and move on.
  If the entire MODULE has no importers (check `search_files` for `import.*<Module>`),
  the sorrie may be in an architectural dead end. Do not invest days in local
  coefficient hunting for a lemma that nothing uses. See `references/dag-module-audit.md`.

- **INCREMENTAL FAMILY EXTENSION**

**For structural invisibility audit** — when a coefficient resists ALL half-Leibniz
T1-T4 and bracketSource positions: enumerate every (source, partner, target) triple
computationally, classify into T1/T2/T3/T4/bracketSource, verify structural invisibility.
**Follow up with a generator audit** (see `references/structural-invisibility-audit.md#generator-audit-follow-up-to-structural-invisibility`)
to confirm the spanning-based closure is sound before formalizing in Lean.
The correct closure is then via spanning/dimension arguments, not new SCC lemmas.
See `references/structural-invisibility-audit.md`.

**Pitfall: structure types need explicit `Zero` instance for `simp`/`abel`/`ring`.**\nWhen a structure (`HalfDerivation`, etc.) has `Add`, `Neg`, `Sub`, `SMul` instances\nbut no `Zero` instance, `simp` with `add_comm`, `abel`, and `ring` all fail because\nthey require `AddGroup` or `Ring` typeclasses that depend on `Zero`.  The error is\n`abel_nf made no progress` or `failed to synthesize OfNat (HalfDerivation F n) 0`.\nFix: add a `Zero` instance at the instance-definition site:\n```lean\ninstance : Zero (HalfDerivation F n) where\n  zero := { coeff := λ _ _ _ _ => 0\n            half_leibniz := by\n              intro ...; simp [bracketSource, bracketIdentity, bracketLeft, bracketRight] }\n```\nThen `abel` and `simp` with `add_comm`/`add_assoc` will work in downstream files.\nDo NOT work around this with manual `calc` proofs — the missing instance breaks\nALL algebraic simplifications, not just the current lemma.\n\n**Pitfall: `induction'` does not generalise dependent hypotheses.**
When adding an extra hypothesis to a theorem that is proved by `induction'` with
`generalizing`, the induction hypothesis `ih` will NOT get a fresh copy of the new
hypothesis unless you either (a) add it to the `generalizing` clause, or (b) `revert`
it before the induction and `intro` it after. Pattern:
```lean
revert h_extra
induction' hlen : v - u using Nat.strong_induction_on with d ih generalizing i j u v
intro h_extra
```
Without this, `ih` calls expecting the extra argument fail with
"Application type mismatch".

**Pitfall: `HalfDerivation` lacks bundled algebraic typeclasses.**
The structure has standalone `Add`, `Zero`, `Neg`, `Sub`, `SMul` instances but
NOT `AddSemigroup`, `AddMonoid`, `AddGroup`, or `AddCommGroup`. Consequence:
`simp` cannot use `add_zero`, `add_comm`, `add_assoc` at the `HalfDerivation`
level; `abel`/`ring` are unusable. Simple identities like
`D = s·Id + (D - s·Id)` must be proved at the coeff level via `ext_coeff`
(see `references/d3-assembly-methodology.md`).

**Most critically: `LinearMap M →ₗ[F] N` requires `AddCommMonoid M` and
`Module F M`.** If you define `coeffOf_map : HalfDerivation F n →ₗ[F] V F`,
it will fail with `failed to synthesize instance AddCommMonoid (HalfDerivation F n)`.
The fix is to add the missing instances in the structure-definition file
(where `variable (F n)` are explicit, so `rfl` works for coeff projections):

```lean
-- In HalfDerivation.lean, after existing Add/Zero/SMul instances:

private lemma coeff_inj {D1 D2 : HalfDerivation F n} (h : D1.coeff = D2.coeff) : D1 = D2 := by
  rcases D1 with ⟨c1, hl1⟩
  rcases D2 with ⟨c2, hl2⟩
  subst h; rfl

instance : AddCommMonoid (HalfDerivation F n) where
  add_assoc D1 D2 D3 := coeff_inj (by ext i j u v; simp [coeff_add, add_assoc])
  zero_add D := coeff_inj (by ext i j u v; simp [coeff_add])
  add_zero D := coeff_inj (by ext i j u v; simp [coeff_add])
  add_comm D1 D2 := coeff_inj (by ext i j u v; simp [coeff_add, add_comm])
  nsmul := nsmulRec

instance : Module F (HalfDerivation F n) where
  add_smul r s D := coeff_inj (by ext i j u v; simp [coeff_add, coeff_smul, add_smul, mul_add])
  zero_smul D := coeff_inj (by ext i j u v; simp [coeff_smul])
  smul_zero r := coeff_inj (by ext i j u v; simp [coeff_smul])
  smul_add r D1 D2 := coeff_inj (by ext i j u v; simp [coeff_add, coeff_smul, mul_add])
  mul_smul r s D := coeff_inj (by ext i j u v; simp [coeff_smul, mul_assoc])
  one_smul D := coeff_inj (by ext i j u v; simp [coeff_smul])
```

The `coeff_inj` helper (NOT `coeff_injective` — that lemma is in a downstream file)
proves that `HalfDerivation` equality follows from `coeff` equality (since
`half_leibniz` is a Prop). The `ext i j u v` works because `D1.coeff = D2.coeff`
is Pi equality (4 indices → F), and Pi has a built-in `@[ext]` lemma via `funext`.

Without these instances, ANY use of `LinearMap` on `HalfDerivation` will fail.
Do NOT work around this with manual `calc` — the missing instances block
ALL `LinearMap` operations, not just the current lemma.

**Pitfall: `cases` on a function-equality hypothesis (not `rw`).**
When `h : cA = cB` and both are `ℕ⁴→F`, `rw [h]` fails with
"motive is not type correct" because of dependent type issues with the
`half_leibniz` field. Use `cases h` instead — it substitutes `cA` for `cB`
in the goal and all hypotheses, avoiding the motive problem.

---

## 🔷 Paper Proof Architecture — Writing Discipline

When the user transitions from proof development to paper writing, enforce
strict section-boundary discipline. The paper's constraint-reduction
architecture (L0–L7 in `docs/PROOF_ARCHITECTURE_FINAL.md`) maps to
sections with non-overlapping responsibilities:

```
§5 — CONSTRAINT REDUCTION
  Only proves reduction mechanisms.
  Never counts dimensions (n+5).
  Never names basis vectors (B, C, D, E, F).
  Never states the main theorem.
  §5.6 ends with: characterization complete.

§6 — INTERPRETATION
  Only interprets the constraint system.
  No proof obligation. No new mechanisms.

§7 — ASSEMBLY
  Only assembles results from §3 and §5.
  No new mechanism. No new lemmas.
  Proof fits on one page.
```

**CRITICAL PITFALL: "self-pairing forces coefficients to zero" is a mathematically WRONG proof.**
The equation `[T(E),E] + [E,T(E)] = 0` is identically true by antisymmetry
of the Lie bracket (`[A,B] = -[B,A]`), regardless of T. It provides NO
constraint on the coefficients of T(E). Any proof sketch that claims to
deduce `x_{pq}=0` from this equation alone is incorrect — expanding in basis
gives Σ x_{pq}·([E_{pq},E]+[E,E_{pq}]) = Σ x_{pq}·0 = 0, which is true
for any coefficients. The actual image restriction proof in Lean
(`centered_image_in_I`) uses classification lemma dispatch (BoundaryRigidity,
AdjacentVNZero, offdiag_zero_v_lt_n, etc.), not a simple self-pairing
expansion. For lecture notes or paper sketches, either (a) defer to the
Lean verification, or (b) give the correct three-case dispatch (diagonal→
centeredness, v=n→adjacent_vn classification, v<n→width filtration).
NEVER present the fake self-pairing argument as a proof.

**Key pitfall: self-pair is NOT a major constraint mechanism.**
For centered T₀ at adjacent source (k,k+1), the self-pair half-Leibniz
condition `[T₀(E_k),E_k] + [E_k,T₀(E_k)] = 0` is **vacuous for interior
sources** (k=2,…,n-2). The a-, b-, c-channel components of T₀(E_k) bracket
to zero with E_k for interior k. Only at k=1 does it give `c₁=0`, and at
k=n-1 the terms cancel. Self-pair contributes at most 1 constraint, not
n-1. The **image restriction Im(T₀)⊆I** is the dominant rank contributor
(~55% of total constraint rank for n=5). Do NOT claim self-pair eliminates
n-1 degrees of freedom.

**CRITICAL PITFALL: "self-pairing forces coefficients to zero" is mathematically WRONG.**
The equation `[T(E),E] + [E,T(E)] = 0` is identically true by antisymmetry
of the Lie bracket (`[A,B] = -[B,A]`), regardless of T. It provides NO
constraint on the coefficients of T(E). Any proof sketch that claims to
deduce `x_{pq}=0` from this equation alone is incorrect. The actual
image restriction proof in Lean (`centered_image_in_I`) uses the full
classification dispatch (BoundaryRigidity, AdjacentVNZero, etc.),
not a simple self-pairing expansion. For lecture notes or informal
explanations, give geometric intuition ("only extreme upper-right
corner elements survive") and honestly note that the rigorous proof
is more involved, rather than presenting a fake simple argument.

**Pitfall: n=4 is exceptional.** For n=4, diagonal propagation fails
because [E₁₂,E₂₄]=E₁₄ and [E₁₃,E₃₄]=E₁₄ create independent constraints,
leaving 2 diagonal DOFs instead of 1. `dim HalfDer(N₄) = 10 = n+6`, not
n+5. The main theorem must state n≥5, with n=4 as a separate remark.
Always verify boundary cases (n=4, n=5) computationally before claiming
a formula holds for "all n".

**Pitfall: Paper↔Lean correspondence requires proof-chain tracing.**
Do NOT match theorem names only. For each paper claim, trace the Lean
proof chain step-by-step. Example: the paper's F_eq `a₁+c_{n-1}=0`
corresponds to `F1_coeff_relation` (p=1 branch) + `F2_coeff_relation`
(p=n-1 branch) in Spanning.lean, NOT `boundary_rigidity` (which proves
`coeff(i,i+1;i,n)=0`, a different statement). The Lean proof chain is:
`commuting_adj_F1 → bracketSource_commuting_eq_zero → Phi=0 →
bracketIdentity_at_1n_canonical → indicator expansion → a₁+c_{n-1}=0`.
Always verify that the Lean theorem's conclusion matches the paper's
after channel decomposition, not just by theorem name similarity.

**Pitfall: "modulo channel-coupling terms" is hand-wavy.** When a paper
proof says "modulo channel-coupling terms from the a-, b-, and c-channels,"
replace it with an explicit index analysis: for interior k, E_{1,n} is
central, E_{1,n-1} only brackets with partner index n-1 or n, and E_{2,n}
only brackets with partner index 1 or 2 — none of which occur for interior k.
The endpoint cases (k=1, k=n-2) are verified separately.

**For paper translation (English → Chinese)** — the 10-principle methodology,
two-phase workflow, glossary-first approach, Chinese mathematical writing
conventions, and per-section freeze discipline established during the `e/`
project paper translation: see
  `references/chinese-paper-translation-workflow.md`.

---

## 🔷 Six-Layer Project Philosophy

Every complex proof project decomposes into layers. Each layer answers
exactly one mathematical question. Layers must not be mixed.

| Layer | Question | Typical Work |
|-------|----------|-------------|
| Result | Why is the answer not what we thought? | Identify the nature of the error |
| Object | What are we studying? | Phase 0: object diagram, spaces, morphisms |
| Language | Why do Paper and Lean look different? | Correspondence audit |
| Proof | What actually has to be proved? | Isolate obligations into a DAG |
| Presentation | Why should the reader find this inevitable? | Narrative organization, readability audit |
| Method | How should we begin next time? | Reusable workflow |

**The Principle.** Every layer should answer exactly one mathematical question.
A proof project fails when layers are mixed (computing before defining objects,
proving before establishing language, writing before isolating obligations).

**The Workflow.**
1. Fix the objects (spaces, maps, commutative diagram).
2. Fix the language (Paper–Lean correspondence).
3. Isolate the proof obligations (manifest with DAG).
4. Discharge the obligations (prove, correspond, or declare vacuous).
5. Organize the narrative (hide Phase structure behind Sections).
6. Audit for independent readability.

### Execution Rule (after architecture freeze)

Once Phases 0–F are frozen:
- No new mathematical objects.
- No new architectural phases.
- No modification of 0–F unless a logical contradiction is discovered.
- All new work is either (1) proving a deferred lemma, or (2) assembling
  previously proved results.

### Conjecture Dependency Rule

Any Phase E/F/G reference to a Conjecture (Phase D §D.3.x) MUST be
explicitly marked "Assuming Conjecture D.3.x." Conjectures may not be
cited as established facts.

### Proof Status Reporting

Never report proof progress as a percentage. Report exact counts:
```
Established:  N
Core Gaps:    M
Sketch:       K
```
Percentages mislead because one lemma may be nearly complete while
another hides an undiscovered structural obstacle. Counts are objective.

### Three Status Labels Only

Mathematical claims must use exactly one of: **Established**, **Conjectured**,
**Conditional**. Never "Key finding," "Key observation," "Expected,"
or percentage-based progress.

### The Two Outputs

Every project following this method produces:
1. **A mathematical result.** The theorem and its proof.
2. **A method result.** The proof architecture — reusable for future projects.

Full methodology reference: `references/foundational-specification-methodology.md`.

**Expansion-first methodology — compute before conjecturing.**
When designing a proof strategy for a propagation or syzygy claim, do NOT
start by conjecturing a formula like `φ_i = 2·φ_{i+1}`. Instead, pick a
concrete small parameter set (e.g. i=1,j=3,l=4), expand Ψ fully into
coefficient form, and check whether the formula even holds. Raw expansion
often reveals that the conjectured form is wrong while the Lean code's
mechanism (e.g. `coeffOf_cond` split-k recursion) was correct all along.
See: `references/lean-paper-correspondence.md`.
See: `references/lean-paper-correspondence.md`.