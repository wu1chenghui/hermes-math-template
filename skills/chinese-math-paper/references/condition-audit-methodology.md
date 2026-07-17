# Paper-vs-Lean Condition Audit Methodology

> Applied 2026-07-11 to paper v3 against Lean formalization.
> Initial per-lemma audit found missing condition statements.
> Resolution: global declaration approach (matching KK's style).

## Method

For each theorem/lemma in the paper, check three things:

1. **What the paper states** — the explicit hypotheses in the lemma statement
2. **What the proof uses** — scan the proof body for division by 2 (→ char≠2),
   division by 3 (→ char≠3), use of intermediate indices (→ n≥k), or reliance on
   dependent lemmas
3. **What Lean requires** — the hypotheses on the corresponding Lean theorem

## Key Lean theorems and their hypotheses

| Lean theorem | File | Key hypotheses |
|-------------|------|---------------|
| `all_diag_equal` | HalfClassification | CharNeTwo F, n≥5 |
| `centered_diag_zero` | Centering | CharNeTwo F, n≥5 |
| `centered_image_in_I` | Centering | CharNeTwo F |
| `boundary_rigidity` | BoundaryRigidity | n≥5 (via index ranges) |
| `finrank_kerPhi_eq_n_plus_four` | Spanning | n≥4 |
| `finrank_halfDer` | Centering | n≥5, (3:F)≠0 |

char≠3 is only used in Lemma 3.2 Case 2(v) (3X=0 → X=0). char≠2 is used
in Lemma 1.2 Step 2 (d₁=d₂) and Lemma 3.1 (u=1 case: 4Y=2Y → 2Y=0 → Y=0).

## Solution: Global condition declaration

Rather than repeating conditions in every lemma, add a single declaration
after the main theorem statement:

```latex
\begin{theorem}\label{thm:main}
For n≥5 and charF≠2,3, dim Δ(N(n,F)) = n+5.
\end{theorem}

In what follows we work under the hypotheses of Theorem~\ref{thm:main}.
Thus charF≠2,3 and n≥5 throughout.
```

This matches KK's style: KK declares charF=0 once and all subsequent lemmas
inherit it. Advantages:
- Cleaner lemma statements (no repeated conditions)
- Matches standard math paper conventions
- Eliminates the risk of a lemma forgetting to state a condition

## Per-lemma condition verification (post-fix)

After adding the global declaration, verify that every lemma's proof
references are covered:

| Lemma | What proof uses | Covered by global? |
|-------|:---:|:---:|
| L1.1 | (none special) | ✓ |
| L1.2 | n≥5 (Step 2), char≠2 (½) | ✓ global |
| L1.3 | n≥5 (via L1.2) | ✓ global |
| Prop1.2 | (none special) | ✓ |
| L3.1 | char≠2 (line 46) | ✓ global |
| L3.2 | char≠2,3 | ✓ global (conditions removed from statement) |
| L3.3 | n≥5 (via φ₀) | ✓ global |
| Thm1.1 | n≥5, char≠2,3 | ✓ exact |
