# Interface Verification Before Wiring

When an existing theorem is claimed to close a sorrie, verify the interface
BEFORE editing the file. Trust no summary — only LSP output.

## When to use

- A new theorem was just completed in a separate file
- AGENTS.md / NEXT SESSION ENTRY says "close sorrie X using lemma Y"
- Any agent (human or AI) claims "just import X and apply it"
- The theorem and the sorrie live in different modules with different imports

## The 3-dimension check

Load the theorem signature via `lean_file_outline` or `lean_term_goal`, and
the sorrie goal via `lean_goal`. Then compare:

### Dimension 1 — Hypothesis compatibility

Does the theorem require preconditions the sorrie context doesn't provide?

| Theorem needs | Sorrie has | Compatible? |
|--------------|-----------|-------------|
| `D : HalfDerivation F n` (any) | `D' : HalfDerivation F n` (centered) | ✅ Theorem applies to D' |
| `3 ≤ i` | `1 ≤ i` | ❌ i=1, i=2 not covered |
| `i + 2 ≤ n` | `i < n` generally | ❌ i=n-1 not covered |

### Dimension 2 — Object compatibility

Does the theorem operate on the same object type as the goal?

| Theorem object | Sorrie goal object | Compatible? |
|---------------|-------------------|-------------|
| `D.coeff` (raw coefficient function) | `D'.coeffOf` (validated wrapper) | ❌ Need bridge: `coeffOf_f` at valid indices |
| `HalfDerivation F n` | `(fun i j u v => coeffOf ...)` | ❌ Need lambda application |

### Dimension 3 — Conclusion compatibility

Does the theorem's conclusion match the goal shape?

| Theorem concludes | Sorrie needs | Compatible? |
|------------------|-------------|-------------|
| `coeff i (i+1) i n = 0` | `coeffOf i j u n = 0` for ALL j | ❌ Only covers adjacent j=i+1 |
| Fixed `u=i` | Arbitrary `u` (off-diagonal allows `u≠i`) | ❌ Only covers u=source_row |
| Equal to 0 | Equal to 0 (needed for `False` by contradiction) | ✅ (when indices match) |

## Case study: BoundaryRigidity ↔ Centering:390 (2026-06-28)

### What summaries claimed

> "BoundaryRigidity.lean is complete (0 errors, 2953 jobs). Import it and
> `boundary_rigidity` closes the v=n sorrie at Centering.lean:390."

### What interface check revealed

**BoundaryRigidity theorem:**
```lean
theorem boundary_rigidity (i : ℕ) (hi : 3 ≤ i) (hn : i + 2 ≤ n) :
    D.coeff i (i+1) i n = 0
```

**Centering.lean:390 goal (in `by_contra! h_notI`):**
```lean
D' : HalfDerivation F n := D - scalarCoeff D • halfDeriv_Id
hvn_eq : v = n
h_diag : ¬(u = i ∧ v = j)     -- off-diagonal, source NOT restricted to adjacent
hcoeffOf_ne : D'.coeffOf i j u v ≠ 0
⊢ False
```

**Actual gap:** BoundaryRigidity covers only `(i, i+1, i, n)` — adjacent source,
target row = source row, `3 ≤ i ≤ n-2`. The sorrie requires `(i, j, u, n)` for
ARBITRARY `j > i` and ARBITRARY `u` (with off-diagonal constraint). 

Three mismatches:
1. Source width: adjacent only vs. arbitrary
2. Target row: u=i only vs. arbitrary off-diagonal u
3. Index range: 3≤i≤n-2 vs. all i≥1

**Verdict:** BoundaryRigidity is necessary but NOT sufficient. Additional machinery
(Spanning, kerΦ dimension argument, or generalization of the syzygy to non-adjacent
sources) is needed for the full v=n closure.

## Decision tree after interface check

```
All 3 compatible?
├─ YES → wire immediately (exact boundary_rigidity D' i hi hn)
└─ NO  → classify gap:
         ├─ Missing bridge (Dimension 2): add lemma, not new math
         ├─ Partial coverage (Dimension 1 or 3): theorem solves a SUBSET
         │   └─ Identify remaining subsets and their closure paths
         └─ Wrong theorem entirely: search for correct closure path
```

## Tool sequence

```bash
# 1. Get theorem signature
lean_file_outline(file_path="E/Classification/BoundaryRigidity.lean")
# Look for: boundary_rigidity start_line, end_line, type_signature

# 2. Get sorrie goal
lean_goal(file_path="E/Classification/Centering.lean", line=390)
# Inspect: goals_before (hypotheses + ⊢ goal)

# 3. Compare on 3 dimensions
```

## Common false-confidence patterns

- **"The file compiled with 0 errors"** — Compilation only means the module is internally consistent, not that its theorems match external goals.
- **"NEXT SESSION ENTRY says wire them together"** — NEXT SESSION entries are plans, not verified interface contracts.
- **"The theorem name matches the concept"** — `boundary_rigidity` sounds right, but name ≠ signature compatibility.
- **"Same indices appear in both"** — `coeff i (i+1) i n` appears in both, but the sorrie has GENERAL `j` not fixed to `i+1`.
