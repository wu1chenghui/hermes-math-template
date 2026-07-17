# Assembly Audit Methodology

When a Lean proof has been decomposed into local lemmas that are individually
proven (0 sorries, 0 errors), there remains a structured verification step BEFORE
claiming "the mathematics is complete": the **assembly audit**. This checks that
the local lemmas can actually be wired together to close the target theorem's
proof state.

## When to use

- After a theorem's proof body has been written (even if it has compile errors)
- BEFORE claiming "math is done, only Lean wiring remains"
- When a proof skeleton dispatches to multiple local lemmas via `by_cases`/`cases`

## The three questions

### Question 1: Dispatch tree

For every branch of the target theorem (every `by_cases`, `cases`, `rcases`),
trace the terminal leaf to its closing theorem. Build a tree:

```
target_theorem
├── by_cases A
│   ├── → lemma_X            ← ✅
│   └── by_cases B
│       ├── → lemma_Y        ← ✅
│       └── → lemma_Z        ← ✅
└── → lemma_W                ← ✅
```

Every leaf must map to an EXISTING theorem whose signature is already known.

### Question 2: Coverage gaps

Are there any leaves that do NOT map to any existing theorem? If yes, the
dispatch is incomplete — a new lemma or case analysis is needed. This is
the most important finding an assembly audit can surface.

### Question 3: Interface mismatches

For each leaf→theorem edge, check three dimensions:

1. **Type compatibility**: does the theorem operate on `coeff` or `coeffOf`?
   Does the leaf provide the right representation?
2. **Hypothesis compatibility**: does the leaf context provide all hypotheses
   the theorem requires (validity proofs, characteristic conditions, bounds)?
3. **Section variable alignment**: does the theorem's `(D : HalfDerivation ...)`
   match the leaf's `D'` (which may be `D - scalarCoeff·Id`)?

## Compile errors vs sorries

A CRITICAL distinction:

- **sorries** (`sorry`) = missing proof steps → mathematical gap
- **compile errors** (type mismatch, tactic failure) with 0 sorries = tactical
  issues in an otherwise complete proof

A file with 0 sorries and N compile errors is fundamentally different from a
file with 1 sorrie and 0 errors. The former has a complete proof structure that
needs tactical fixes; the latter has a missing proof obligation.

## Precision rule for status claims

When reporting proof completion status, avoid over-statements:

| WRONG | RIGHT |
|-------|-------|
| "Mathematics complete, only wiring remains" | "All local lemmas proven. Assembly audit shows dispatch tree is complete. N tactical errors remain." |
| "Adjacent v=n dispatch is ✅" | "Adjacent v=n local lemmas are ✅. Coverage in centered_image_in_I verified via assembly audit." |
| "Just Lean wiring left" | "Proof structure complete (0 sorries). N compile errors to fix, all tactical." |

The assembly audit is the bridge between "local lemmas done" and "target theorem
closure verified." Do not skip it.
