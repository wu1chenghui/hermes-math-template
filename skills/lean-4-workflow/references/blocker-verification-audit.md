# Blocker Verification Audit — Before Attacking the Last LIVE Sorrie

When the sorry count drops to 1 and it's been resisting multiple approaches,
STOP. Do NOT iterate tactics, add machinery, or try another partner. First run
this audit to determine whether the sorrie actually blocks anything real.

## When to use

- Single-digit sorries remaining, especially **exactly 1 LIVE sorrie**
- Sorrie has resisted ≥ 2 rounds of independent attacks
- Occurs at a boundary case (v=n, i=1, u=1, etc.)
- Claim: "structurally invisible to half-Leibniz" / "can't be reached by brackets"

## The audit (4 steps, ~15 minutes)

### Step 1 — Are downstream theorems REAL or PLACEHOLDERS?

Search for callers. If the only consumers are `True := trivial` placeholders,
the dependency is a PLAN, not a requirement.

```bash
search_files(pattern="downstream_theorem_name", target="content", path=E/)
```

Check each caller:
- `trivial` / `True := by trivial` → PLACEHOLDER (can be refactored)
- Actual proof body calling the lemma → LIVE dependency

**Critical finding (2026-06-28):** `halfDer_decomposition` and `finrank_halfDer`
were both `True := trivial` placeholders. Their declared dependency on
`centered_part_mem_kerPhi` was a design intent, NOT an implemented fact.

### Step 2 — Map actual code dependencies, not comment dependencies

Read the enclosing lemma's proof body. Trace what it ACTUALLY calls at the
`have`/`apply`/`rw` level — not what the docstring says it needs.

Count the conditions. In `centered_part_mem_kerPhi`, the three `IsKerPhi`
conditions broke down as:
- `IsValidSupp` ← AUTO (coeffOf handles it)
- `Phi = 0` ← AUTO (halfDerivation_phi_zero + linearity)
- `I_filtered` ← SOLE BLOCKER (calls `centered_image_in_I` → v=n)

Knowing that 2/3 conditions are automatic and only 1 is blocked changes the
problem from "can't prove centered_part ∈ kerΦ" to "can't prove I_filtered
for the v=n coefficient."

### Step 3 — Cross-reference the paper's proof

Open the canonical paper documents (`FINAL-THEOREM.md`, D3 blueprint). Check:
- Does the paper's proof ALSO go through this exact lemma?
- Or does the paper use a different structural decomposition?

In the D3 audit: FINAL-THEOREM.md §"Why (b) is well-defined" DOES go through
"Image Containment → T₀ ∈ ker(Φ)". The Lean implementation is faithful to the
paper at this node — the sorrie is mathematically load-bearing, not an artifact.

Also check: does the D3 blueprint (§5) suggest an intermediate layer (HDspace)
that could restructure the proof without changing the mathematics?

### Step 4 — Classify the blocker: math vs interface vs design

| Type | Meaning | Action |
|------|---------|--------|
| **Math** | The statement is a genuine gap in the paper's proof at this exact point | Need new mathematical argument (e.g., dimension counting for v=n) |
| **Interface** | The lemma is needed but the current API is too strong (GAP-A: IsValidSupp) | Redesign the interface (coeffOf solved GAP-A) |
| **Design** | A different proof route bypasses this lemma entirely | Refactor; do NOT attack the sorrie |

The v=n case is **Math**: half-Leibniz structurally cannot reach `coeff(i,i+1;i,n)`,
but `I_filtered` (and hence image containment) requires proving it's zero.
The closure path must be a dimension argument at the assembly level, not a
pointwise coefficient proof.

## Common false signals

- **"The comment says X needs Y"** — Comments lie. Read the proof body.
- **"There's only 1 sorrie"** — Check whether the 1 sorrie is in a live module or an orphan.
  ImageContainment had 7 orphaned sorries + 1 live one. Same count, opposite meaning.
- **"The paper proves this"** — The paper may prove a different formulation. Check
  whether the paper's proof actually handles the edge case (v=n) or assumes it away.

## Output

A 4-line verdict:

```
LIVE sorrie:  centered_image_in_I v=n  (Centering.lean:390)
Blocker type: MATH (structurally invisible to half-Leibniz)
Paper path:   YES — FINAL-THEOREM §(b) requires image containment
Closure:      Dimension argument at D3/D4 assembly, not pointwise proof
```

If the answer is DESIGN (can bypass), refactor IMMEDIATELY — do not attack
the sorrie. If MATH, design the dimension argument before writing Lean.

### Step 5 — Experimental truth-value disambiguation (World A vs B)

When blocker is classified as MATH: before designing a proof, verify the
statement is actually TRUE. Build the full linear constraint system without
the condition you're testing, compute nullspace, compare with known answer.

```
World A (dim matches) → statement is true → need new proof
World B (dim larger)  → statement is false → theorem needs revision
```

Methodology and reusable script: see `references/experimental-nullspace-verification.md`
and `scripts/compute_nullspace_dim.py`.

The 2026-06-28 Boundary Rigidity experiment (n=5,6,7) confirmed World A:
dim(C) = n+4 = dim(kerΦ). The global syzygy phenomenon was discovered —
constraint rows collectively force the coefficient to zero even though
each individual equation is trivial.
