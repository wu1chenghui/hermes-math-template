# Project Charter Structure

When summarizing a mature proof project for future agent sessions, the
AGENTS.md header (or a standalone PROJECT_CHARTER.md) should contain these
six components. The goal is to prevent agents from re-litigating settled
decisions or redesigning frozen architecture.

## 1. Status Block

Four-line status summary — no ambiguity:

```
Research            COMPLETE
Architecture        FROZEN  (Phase 0–F)
Proof               CLOSED  (D′: 0 blockers)
Writing             IN PROGRESS
```

## 2. Current Goal

Explicitly state that the remaining work is **not mathematical discovery**.
List concrete exposition-level tasks:

1. strengthening exposition
2. improving logical flow
3. removing unnecessary material
4. increasing independent readability
5. preparing the paper for peer review

One sentence: "No new mathematics is expected."

## 3. Success Criterion

A concrete, falsifiable statement of what "done" means. Example:

> The project is considered complete when an independent referee can read
> Sections 2–7 of the paper without access to the Phase documents and
> understand why the main theorem follows naturally.

## 4. Frozen Contracts

State WHAT is frozen and, critically, **what to do when a problem is found**:

- Phase 0–F are immutable.
- If a problem is found, suspect the proof or the narrative first — **not the object architecture**.
- Do not introduce new Phases, objects, or dependency edges.
- Do not redesign the architecture.
- The only legitimate reason to modify Phase 0–F is discovery of a genuine **logical contradiction** (not a gap, not a weakness — a contradiction).

## 5. Lean–Paper Relationship

Explicitly state that Lean and Paper are two presentations of the **same mathematical mechanism**, not two independent proofs. Include the correspondence identity (e.g. `coeffOf_cond = Ψ(...) = 0`). Rule: keep them synchronized, do not treat as independent artifacts.

## 6. Abandoned Approaches (Anti-Pitfalls)

List disproved conjectures or dead-end routes with a brief explanation of WHY they failed, so future agents don't re-explore them. Example:

> `φ_i = 2φ_{i+1}` (BL-to-BL propagation): disproved by Ψ-expansion for (1,3,4).
> Actual mechanism: `coeffOf_cond` split-k recursion via `Ψ(E_ik, E_kj, T) = 0`.

## Optional: Proof Debt Status

If proof is closed, state it explicitly with the closure breakdown:

```
Established:  N
Vacuous:      M
Sketch:       K  (structural corollaries only)
To prove:     0
```

This prevents agents from thinking there are hidden mathematical blockers.

---

This structure emerged from the Half-Derivation Classification project
(2026-07-02) after a session where the agent's project summary was rated
90–95/100 — strong but missing these six governance signals.
