# Lean ↔ Paper Correctness Audit Workflow

> Distilled from the 2026-07-12 paper-v3 audit session.
> After every round of paper edits (compression, translation,
> restructuring), verify all mathematical claims against the
> Lean formalization. This catches condition drift, deleted
> hypotheses, and misstated conclusions.

## Prerequisites

- Lean project at `/opt/lean-home/lean-projects/e/E/Classification/`
  (or wherever the user's project lives)
- Paper source in `/workspace/paper-v3/` (English canonical)
- All 11 Lean files compiled with 0 sorries

## Workflow

### Step 1: Extract all Lean theorem signatures

```python
import re, os
lean_dir = "/opt/lean-home/lean-projects/e/E/Classification"
for fname in sorted(os.listdir(lean_dir)):
    if not fname.endswith('.lean'): continue
    with open(os.path.join(lean_dir, fname)) as f:
        text = f.read()
    for m in re.finditer(r'((?:theorem|lemma)\s+(\w+)\s*[:\n(])', text):
        name = m.group(2)
        # extract full signature
```

### Step 2: Map paper claims to Lean theorems

Build a one-to-one mapping table:

| Paper claim | Lean theorem | File | Conditions check |
|------------|-------------|------|:--:|
| Lemma 1.1 (width reduction) | `coeffOf_cond_of` (iterated) | Core | char≠2 ✓ |
| Lemma 1.2 (diagonal equality) | `all_diag_equal` | HalfClassification | n≥5 ✓ |
| Lemma 1.3 (centered decomp) | `halfDer_decomposition` | Centering | n≥4, char≠3 ✓ |
| Lemma 3.2 (im ⊆ I) | `centered_image_in_I` | Centering | n≥5, char≠3 ✓ |
| Lemma 3.3 (endpoint a_k=0) | `interior_a_zero` | Spanning | 2≤p≤n-3 ✓ |
| Lemma 3.3 (endpoint c_k=0) | `interior_c_zero` | Spanning | 3≤p≤n-2 ✓ |
| Lemma 3.3 (coupling) | `coupling_a1_cn1` | Spanning | n≥4 ✓ |
| Theorem 1.1 (dim=n+5) | `finrank_halfDer` | Centering | n≥5, char≠3 ✓ |

### Step 3: Verify condition alignment

For each Lean theorem, check its hypotheses against the paper's
stated conditions (global or per-lemma):

- `[CharNeTwo F]` — must have char≠2 somewhere
- `hn5 : 5 ≤ n` — must have n≥5
- `h3 : (3 : F) ≠ 0` — must have char≠3
- `hI : I_filtered ...` — must have established the I-filtered property

Paper uses: "Throughout, char F≠2,3 and n≥5."

### Step 4: Verify conclusion alignment

Check that the Lean conclusion means the same thing as the paper's
statement. Key translations:

| Lean notation | Paper notation |
|--------------|---------------|
| `coeffOf D i j u v` | π_{uv}(φ(E_{ij})) |
| `scalarCoeff D` | the scalar s in φ = s·id + φ₀ |
| `I_target_set` | I = {(1,n-1), (1,n), (2,n)} |
| `Module.finrank F ...` | dim |

### Step 5: Run Lean build to confirm 0 sorries

```bash
cd /opt/lean-home/lean-projects/e && source ~/.elan/env && lake build
```

## Common Pitfalls

1. **Comment `\ref` mismatch**: English has `% --- Main proof of Lemma~\ref{lem:imageinI} ---`
   as a comment; Chinese lacks it. This causes a ref-count difference but
   is harmless — it doesn't enter the PDF.

2. **Condition strength**: Paper may claim "n≥5" while Lean only needs "n≥4"
   for some lemmas. This is fine — the paper's condition is *stronger*.

3. **Lemma 1.1 (width reduction)**: Lean doesn't have a single theorem for
   this. The principle is embedded in `coeffOf_cond_of` and used iteratively
   throughout proofs. The paper's statement is a valid generalization of
   this principle.

4. **Lemma 3.1 (Claim)**: Lean splits this into multiple theorems
   (`adjacent_rowcol_zero`, `adjacent_above_zero`, `adjacent_below_zero`,
   `adjacent_vn_zero`) covering different u ranges.

## After Audit

- [ ] All 16 paper claims mapped to Lean theorems
- [ ] All conditions verified: char≠2, char≠3, n≥5 across all lemmas
- [ ] No Lean theorem requires a condition the paper doesn't state
- [ ] `lake build` returns 0 sorries
