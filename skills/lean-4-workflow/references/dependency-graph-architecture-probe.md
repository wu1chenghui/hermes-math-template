# Dependency-Graph Architecture Probe (GATE-D/E/F/G pattern)

A methodology for discovering the well-founded measure and API shape of a proof
from the coefficient-level dependency graph, BEFORE writing Lean skeleton code.
Emerged from the D3 imageContainment architecture discovery (June 2026).

## When to use

When a theorem's proof architecture is genuinely unknown — the induction
measure, the dispatch structure, and the leaf-lemma API are all open
questions — and you need computational evidence to settle them.

## The five-probe sequence

### GATE-D: SCC decomposition of the naive dependency graph

Build the dependency graph using the CANONICAL mechanism per node
(split-descent for nonadjacent, best commuting-partner for adjacent).
Run Tarjan SCC. The nontrivial SCCs (size > 1) are the cyclic clusters
that CANNOT be closed by single-pin per-coeff lemmas.

Key question: are the SCCs bounded (size ≤ constant) or growing with n?

### GATE-E: Full move-set SCC size bound

Add ALL prospective moves (commuting-partner for wide∈I/nonadjacent nodes,
not just adjacent). Rerun schedulability fixpoint + greedy (min-in-core-dep
per node) SCC. This gives the ACHIEVABLE cluster decomposition — the
best-case joint-closure units under optimal move assignment.

The greedy max-SCC answers: bounded (finite lemma family) vs growing with n
(parameterized closure). BUT: must test n large enough — small-n artifacts
can hide growth (observed: n≤8 showed maxSCC≤3; n=9,10 revealed true
growth to ~n−5).

### GATE-F: Shape stabilization census

Use a CANONICAL FORM for cluster shapes that is n-independent:
- Anchor boundary indices as L1, L2, R0, R1
- Rank middle indices within the cluster as M0 < M1 < ...
- Record adjacency pairs (which indices are consecutive)

Count distinct shapes at n₁, n₂, n₃. Check whether |Tₙ| stabilizes.
If it does: the coupling-lemma API is a FINITE set of parameterized
templates. If it keeps growing: the clusters are genuinely n-dependent
(parameterized chain-induction needed).

### GATE-G: Mechanism classification completeness

Classify every SCC (at the largest tested n) into mechanism families.
For D3 the families were: width_c_chain, width_a_chain, internal_det_coupling.
Check: no SCC falls outside these families (no 6th mechanism).

### part-2: det/rank per cluster (char-dependence)

For each cluster, build the LOCAL linear system (the cluster-internal
relations: coeffOf_cond splits + commuting-partner bracketIdentities,
restricted to cluster members with externals = 0/schedulable). Compute
rank mod p (p=3,5,7,13). Determines:
- Whether char≠3 closes all clusters jointly (rank=full for p∈{5,7,13})
- Whether char=3 is degenerate (rank < full for p=3)
- Sparsity: nonzeros-per-relation (=2 means chain/bidiagonal, closed by
  telescoping induction; >2 means dense k×k system)

The sparsity result determines whether the chain closure is a parameterized
CHAIN-INDUCTION (one lemma per channel, cheap) vs a dense k×k cluster
formalism (heavy).

## Implementation pattern

All probes are **non-landed** (Python `execute_code` or Lean `lean_run_code`
snippets only). No project files are modified. The probes answer
architecture questions, not proof questions.

The computational model is combinatorial: edges encode the dependency
structure of the proof mechanism (split children, commuting-partner
residuals). The model must be FAITHFUL to the actual Lean coefficient
relations (bracketIdentity T1-T4, coeffOf_cond arms, I_target_set).

## What the probes produce

After GATE-G completes:
- A FROZEN set of mechanism families (e.g., 3: width_c, width_a, internal_det)
- Per-family closure type (chain-induction vs finite lemma vs coupling)
- The global induction measure (cluster-DAG topological rank vs target-width)
- Char-dependence (which families are char≠3-sensitive, which are universal)

This directly determines the skeleton's dispatch structure and the
parameterized lemma API, making the skeleton "discovered" rather than
"designed."
