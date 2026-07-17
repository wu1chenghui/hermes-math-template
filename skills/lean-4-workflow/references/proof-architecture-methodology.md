# Proof Architecture Methodology

> Established during the foundational re-audit of the HalfDer classification
> project (2026-06-30 / 2026-07-01). This is a general methodology for
> structuring a mathematical proof project into phases with strict boundaries.

---

## The Three-Layer Model

```
Architecture (0–F)  →  Proof (D′)  →  Assembly (G)
```

- **Architecture**: Define objects, maps, interfaces. Freeze before proving.
- **Proof**: Prove every deferred/conjectured claim. No new objects allowed.
- **Assembly**: Combine proved results into the main theorem. One-line invocations only.

---

## Phase Design Rules

### 1. Each Phase has exactly one Primary Object

Every Phase header declares its primary mathematical object:

```
> **Primary mathematical object:** Bases of SEq — minimal generating sets.
```

All content in the Phase must be organized around this object. No other object
may be the focus.

### 2. Each Phase declares what it does NOT establish

Example:

```
This phase does NOT establish:
- Linear relations among generators (syzygy) — Phase D.
- Rank or dimension of SEq — Phase E.
```

This prevents scope creep and makes audits straightforward.

### 3. Dependency DAG must be acyclic

```
Phase 0 → A → B → C → D → E → F → G
```

No reverse edges. If a dependency cycle is found, split one of the phases.

### 4. Exported Results table

Every Phase ends with:

| Export | Description | Status |
|--------|------------|:---:|
| ... | ... | Established / Conjectured / Conditional / Design |

And a matching "NOT exported" table.

---

## Object Separation Principle

Every mathematical entity belongs to exactly one category:

1. Algebraic object (vector space, Lie algebra)
2. Linear map (morphism between objects)
3. Image (of a map)
4. Kernel (of a map)
5. Coordinate representation (coefficient, matrix entry)
6. Logical proposition (theorem, lemma)

No cross-category identification without an explicit morphism. Never say
"this equation IS this constraint" — say "this functional φ ∈ SEq evaluates
to zero on T."

---

## Conjecture Dependency Rule

When a Phase references a Conjecture from an earlier Phase:

> Any reference to a Conjecture (D.3.x) or Conditional result MUST explicitly
> annotate the dependency: "Assuming Conjecture D.3.x."

Conjectures may never be cited as established facts.

---

## Execution Rule

Once Architecture is frozen:

- No new mathematical objects.
- No new architectural phases.
- No modification of frozen Phases (unless a logical contradiction is discovered).
- All new work is either proof completion or theorem assembly.

---

## Audit Types

### Responsibility Audit

Check that every section only does what the Phase header claims. Look for:
- Phase boundary leakage (e.g., bracket decomposition in a generation catalog)
- Premature conclusions (e.g., "these force all diagonals equal" in a morphology phase)
- New object creation (e.g., introducing a family name that isn't defined in prior Phases)

### Theorem Audit

Classify every claim as Definition / Observation / Conjecture / Theorem.
For each "Theorem"-level claim, check whether a proof exists or is sketched.
If not, downgrade to Conjecture.

### Object Purity Audit

Check that the Phase uses only its declared inputs. Example: Phase F must
reference only Phase A and Phase E objects — no direct references to Ψ or
generator families from Phase B/C.

---

## Interface Contracts

Each Phase that serves as input to later Phases should declare its interface:

```
> **Interface Contract 1 — Inputs.** Phase F references only:
> - B_cand from Phase E.
> - {e_k} from Phase A.
```

This prevents downstream phases from reaching into encapsulated internals.

---

## Freeze Protocol

Before freezing a Phase:

1. Complete the relevant audit (Responsibility or Theorem).
2. Fix all audit findings.
3. Add `> **Status: FROZEN**` stamp with date.
4. State what the Phase establishes and what it defers.
5. Add to the project ROADMAP.md.

After freezing: no modifications unless a logical contradiction is discovered.

---

## Anti-Patterns

| Anti-pattern | Fix |
|-------------|-----|
| "We have covered all cases" | Build a Coverage Matrix with machine-checkable entries |
| "These two witnesses are independent" | Compute the rank of the constraint Jacobian |
| "The graph is connected" | If unproven: "Conjectured: the syzygy graph is connected (conditional on D.3.x)" |
| Using coordinate names as objects | Define basis vectors {e_α}, defer coordinate labels to the coordinate phase |
| Mixing "what exists" with "how it looks" | Separate the catalog phase from the morphology phase |
| Claiming percentage completion | Report counts: Established / Core Gaps / Sketch |
| "Key finding" or "Key observation" | Use precise labels: Definition, Lemma, Theorem, Conjecture |
