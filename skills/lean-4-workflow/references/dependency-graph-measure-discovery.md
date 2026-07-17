# Discovering the Proof-Assembly Induction Measure via Dependency-Graph / SCC Analysis

## When to use
You have a multi-lemma final assembly (e.g. `imageContainment`: "all ∉I coeffs
vanish") whose induction ORDER / well-founded MEASURE is uncertain. Typically
reached AFTER `mechanism-probe-before-architecture-freeze.md` validated each leaf
class and `assembly-interface-skeleton-probe.md` showed the final theorem is NOT a
single target-width strong induction. Do NOT hand-design a `(phase, target-width)`
layering here — the real order can be a non-obvious cluster-DAG. **Discover it
computationally before writing the skeleton.** This is the third probe in the
sequence: single-leaf mechanism → assembly interface → global dependency order.

## The two obstructions that send you here (compiler-confirmable)
A single target-width strong induction over the conclusion class fails when a
base-case leaf must discharge a residual that is:
- **DOMAIN obstruction** — in the EXCEPTION set (e.g. ∈I), so the ih's quantifier
  (`∀ ∉I …`) can never name it; and/or
- **DIRECTION obstruction** — at a LARGER measure (e.g. wide residual width n−2 vs
  leaf target width ≤ n−3), so an ascending-width ih can't reach it.
Confirm both with a tiny non-landed `lean_run_code`: `exact ih … (show residual ∉
ExceptionSet) …` fails on the membership goal (∈ exception ⟹ `⊢ False`) AND on the
width bound (`omega could not prove width_residual < d`). Two independent errors =
the measure is wrong, not the tactic.

## The recipe (Python, faithful to the verified Lean)
1. **Encode the structural rules FROM THE LEAN SOURCE** (not memory): the
   recursion/descent arms (e.g. `coeffOf_source_exists`'s 4-way disjunction with
   exact index conditions and the split `k`); the leaf residual mechanism (e.g.
   `bracketIdentity_eq_expanded`'s positional T1..T4 + guards — verify your
   encoding reproduces a proven leaf like `boundary_rep_A`'s `have hres`); the
   exception/target sets (`I_target_set`).
2. **Node set = exactly the coeffs the proof must establish = 0.** CRUCIAL: exclude
   genuinely-NONZERO coeffs (the center; adjacent-source ideal targets = the
   exceptional cocycles). Modeling a nonzero target as "killable" invents false
   edges and false cycles.
3. **Moves per node = the prover's real FREEDOM**: every valid split `k`
   (nonadjacent) and every valid commuting partner (adjacent). A move is valid only
   if every dependency it produces is itself a killable node (else discard it).
4. **Schedulability fixpoint**: a node is schedulable iff SOME move's deps are all
   already schedulable; rank = 1+max(dep ranks). Iterate to fixpoint. ALL nodes
   scheduled ⟺ an acyclic single-choice schedule exists, and rank IS the
   well-founded measure → a plain strong induction works.
5. **If not all scheduled → Tarjan SCC** on a canonical-move graph. Nontrivial SCCs
   = the irreducible cyclic CLUSTERS; the condensation is a DAG; the well-founded
   measure = condensation topological rank. Report cluster sizes/count + taxonomy.

## After SCC: distinguish dense-k×k from sparse-chain (deck the cluster's real mechanism)

An SCC report gives sizes but NOT the closure mechanism. A size-5 cluster could be a dense
5×5 linear system (heavy: a new k×k formalism for growing k) or a sparse 2-term-relation
chain (medium: a 1-parameter chain induction generalising an existing step lemma).

**Decide with a PART-2 DET/RANK ANALYSIS on the local linear system of the LARGEST cluster:**
1. For each cluster member, collect its IN-CLUSTER relations (split + commuting-partner, drop
   externals=0), build the rows×coeffs matrix.
2. Compute **nonzeros per row** (the sparsity signature) and **rank mod p** for a control
   set of primes (e.g. 3,5,7,13 — the suspect characteristic and control chars).
3. Interpret sparsity: `nonzeros = {2}` = every relation involves exactly two cluster members
   → **bidiagonal/sparse chain**; `nonzeros ≥ k` = dense. A sparse chain closes by
   **telescoping / parameterized chain-induction** (which the existing width-stability step
   lemma already embodies — just needs to be iterated over chain length), NOT by a dense k×k
   matrix solve. A dense cluster needs a genuine cluster-closure formalism.
4. Interpret rank: `rank = size` for pk ∉ {suspect} and `rank < size` for p = suspect → the
   cluster is char-sensitive (det divisible by the suspect prime); `rank = size` for ALL
   tested primes → the cluster closes unconditionally.

The **nonzeros-per-relation** answer directly determines whether the remaining work is
"parameterized chain-induction" (medium, reusing existing descent lemmas) or "dense
k×k cluster formalism" (heavy, new theory). In the former case the cluster's *size*
grows with n, but the *mechanism* stays a single 1-parameter induction per channel.
GATE-A's 2-cycle is just the chain's smallest (terminal) instance.

## What the clusters mean (the deep lesson)
- A **hypothesis-driven lemma** (e.g. `hI` / `I_filtered`) that the final assembly
  must DROP can hide a real dependency cycle. With the hypothesis it short-circuits
  descent (every exception coeff is 0 by assumption); hypothesis-free it must
  descend for real and the cycle surfaces. The hypothesis-free reconstruction is a
  DIFFERENT proof object, not "the same proof minus a hypothesis".
- The hypothesis-free graph often decomposes into SMALL bounded cyclic clusters
  (each = {a residual target + its descent chain + the leaves that close back}).
  These are NOT closable in a linear `A ← B ← C` order; they must be closed JOINTLY
  as a k×k linear system, frequently with a CHARACTERISTIC-DEPENDENT determinant
  (the same reason the theorem is char-split).
- An existing proven 2-cycle "coupling" lemma (a 2×2 det, e.g. char≠3
  `boundary_coupling`) is usually the SMALLEST cluster's joint-closure lemma; its
  σ-mirror is the next smallest. The remaining work is the k×k generalisation — one
  lemma per cluster family. The nullspace fact (these coeffs ARE 0 in the good
  characteristic) guarantees each cluster's joint determinant is nonzero there, so
  the clusters are always closable; only the Lean mechanism remains.
- Therefore a per-leaf "single commuting partner + `frozen ← boundary ← interior`
  orientation" blueprint can be INSUFFICIENT hypothesis-free; the closure unit is
  the CLUSTER, not the single coeff. This supersedes such a blueprint's
  "remaining work = pure engineering" claim.

## Gotchas
- A FIXED split (always `k=i+1`) + an ARBITRARY partner choice produce SPURIOUS
  cycles. Give the model the prover's real freedom (all k; canonical/all partners)
  before declaring a cycle real — a spurious cycle breaks once you pick the
  canonical partner whose residual descends cleanly.
- Recursion CHILDREN are killed by IH (smaller measure), NOT re-proven via leaf
  lemmas. Don't conflate "appears as a recursion child" with "needs a base-case
  leaf lemma"; leaves fire only on the base case at the current measure.
- **Small-n threshold trap**: declaring \"maxSCC bounded ≤ 3\" based on n₀=8 is UNSAFE
  — the growth threshold can be just past the currently-tested range. In one real
  project, maxSCC was 2,2,3,3 for n=5..8 (suggesting bounded), but 4,5 for n=9,10
  (revealing linear growth ~ n−5). Always extend the probe at least TWO n-values
  beyond the largest that gave a new structure, specifically watching maxSCC for
  growth. The cluster *sizes* may grow while the *mechanism* stays a parameterized
  chain-induction — but knowing which demands the n-extension.
- Cross-check: your Python arm/T-formula encoding MUST reproduce a known proven
  leaf (compute its residual; confirm it matches the Lean lemma).
- Stay non-landed: `lean_run_code` + Python only. Do not write the skeleton until
  the measure (cluster-DAG rank + cluster taxonomy) is locked — landing a
  `(phase,width)` skeleton the cluster decomposition then rewrites is the exact
  rework this probe prevents.

## Output that unlocks the skeleton
- the well-founded MEASURE = condensation/cluster-DAG topological rank (state depth);
- the CLUSTER TAXONOMY = the bounded family of joint-closure lemmas needed (sizes,
  channel, which is the already-proven smallest), so the skeleton's cluster-lemma
  interface and induction parameter are DISCOVERED, not designed.
