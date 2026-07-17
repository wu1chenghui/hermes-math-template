# Proof Pitfalls Discovered During v3 Audit (2026-07-11)

Systematic cross-reference between paper and Lean formalization uncovered
these patterns. Each is a recurring class of error.

## Pitfall 1: Chain bridge applied to wrong source

**Symptom**: Using the chain bridge on source (i,j) to zero a coefficient
from a DIFFERENT source (p,q). The chain bridge relates coefficients WITHIN
a single source, not across sources.

**Example** (Case 1a/1b): Needed to zero π_{2,v+1}(φ(E_{v,v+1})) — coefficient from
source (v,v+1). The paper applied chain bridge to source (2,v+1), which gives
info about φ(E_{2,v+1}), not φ(E_{v,v+1}).

**Fix**: Reference the Claim (three-equation chain bridge system, Case 1d)
which covers all u < i, v = i+1 coefficients.

## Pitfall 2: Strategy summary bounds mismatch

**Symptom**: Strategy overview says "c_k=0 for k≥3" but actual proof says
"c_k=0 for 3≤k≤n-2". The summary loses the boundary exception at k=n-1.

**Why dangerous**: Readers skim the strategy first. Wrong bounds make them
compute wrong surviving variables → wrong dimension.

**Fix**: Always copy precise intervals from proof into every summary.
Never abbreviate "3≤k≤n-2" to "k≥3".

## Pitfall 3: Unverified adjectives on ideal properties

**Symptom**: "I is maximal abelian" — "abelian" is true, "maximal" is false
(counterexample: I + span{E_{1,n-2}} remains abelian). Never used in proof.

**Fix**: Only state properties you have verified. If unused, delete entirely.

## Pitfall 4: Corollary restating a definition

**Symptom**: Corollary says "centered φ → π_{ij}(φ(E_{ij}))=0" but the
DEFINITION of centered IS exactly this. The corollary is P→P, with a proof
that circularly invokes the definition.

**Fix**: Delete the corollary. The centered decomposition lemma (1.4) already
does the work of connecting diagonal equality to centeredness.

## Pitfall 5: Forward reference in decomposition proof

**Symptom**: Centered decomposition (Lemma 1.1 in original ordering) references
diagonal propagation (Lemma 3.1). Reader in §1 encounters a proof that depends
on §3.

**Fix**: Reorder sections so diagonal propagation (§1.4) precedes centered
decomposition (§1.5). Width reduction only needs chain bridge, can stay early.

## Pitfall 6: "i.e." — allowed vs forbidden

**Distinction**:
- `= π_{...}, i.e., 2Y = X` — ALLOWED: substitutes defined symbols
- `π = 0, i.e., the diagonal vanishes` — FORBIDDEN: explains math in prose

The skill's forbidden list originally banned all "i.e." — this was too broad.
