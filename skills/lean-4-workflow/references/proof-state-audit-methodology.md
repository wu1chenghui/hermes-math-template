# Proof-State Audit Methodology

> **When to use**: Before attacking a LIVE sorrie in a Lean proof that sits inside
> multiple case splits. The goal is to determine what the sorrie ACTUALLY needs
> (as opposed to what the enclosing theorem statement claims).

## The Three-Step Audit

### Step 1: Interface Check

When a lemma claims to close a sorrie, do NOT trust agent summaries. Verify:

1. **Read the lemma's source file** — confirm it exists, compiles, and has the right signature
2. **Check the sorrie's goal** via `lean_goal` — get the FULL context with all hypotheses
3. **Compare point-by-point**:
   - Hypothesis match? (D vs D', centered vs raw, etc.)
   - Object match? (HalfDerivation vs coeffTable, coeff vs coeffOf)
   - Conclusion match? (Does the lemma's return type directly close the goal?)

Common mismatch patterns:
- Lemma proves `coeff` but goal needs `coeffOf` (need bridge lemma)
- Lemma works on `D` but context has `D' = D - scalarCoeff·Id`
- Lemma has fixed indices (i,i+1,i,n) but goal has generic (i,j,u,v)

### Step 2: Proof-State Trace

Read the ENTIRE proof from the theorem statement down to the sorrie. Map every
case split:

```
by_cases A → case true: how closed?
           → case false: 
             by_cases B → case true: how closed?
                        → case false: SORRIE ← what's in context here?
```

Key question: **what has already been eliminated by earlier case splits?**

Often the sorrie looks general (∀ i,j,u,v) but the context has already narrowed it
to a specific regime (e.g., j=i+1, u≥3, v=n).

### Step 3: Gap Map (Freedom DAG)

Classify ALL remaining cases by whether they REDUCE to already-proven cases.

Categories:
- **Class I**: Directly handled by existing lemmas
- **Class II**: Reducible via existing reduction machinery (width induction, coeffOf_cond, etc.)
- **Class III**: Not reducible — TRUE remaining gaps

Draw the reduction graph: if case A reduces to case B, draw A→B.
The SINKS of this graph are the true blockers.

## The "Theorem Too Strong" Anti-Pattern

A recurring issue in this project: theorem statements inherited from earlier
proof architectures that are MONOLITHIC (one theorem covering all cases) when
a DECOMPOSED approach is now available.

Signs:
- A single `by_cases` branch that handles a huge class (e.g., "v=n" for ALL i,j,u)
- Comments saying "requires wiring through Spanning" or "structurally invisible"
- New lemmas (like BoundaryRigidity) that only cover a subset

Resolution: do the gap map FIRST. Often the monolithic case decomposes into:
- Already-proven sub-cases (via existing reduction machinery)
- A few narrow sub-cases that need new proofs
- Cases that don't actually arise (vacuous in context)

## Case Study: Centering.lean:390

The sorrie at `centered_image_in_I` line 390 appeared to require proving
ALL `coeff(i,j;u,n) = 0` for v=n, u≥3.

After proof-state audit:
- Diagonal cases: eliminated by earlier `by_cases h_diag` → `centered_diag_zero`
- Non-adjacent sources: reduced via `coeffOf_nonadjacent` (induction on v-u, works for ALL v including v=n)
- Adjacent, u=i: handled by BoundaryRigidity + centered_diag_zero (for i=n-1 boundary)
- Adjacent, u≠i: the TRUE gap — 3 remaining patterns in `adjacent_noncentral_offdiag_zero`

What looked like "all (i,j,u,n)" was actually just 3 narrow cases.
