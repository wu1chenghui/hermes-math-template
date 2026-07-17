# Adjacent-Source v=n Five-Class Dispatch

For a half-derivation D (coeff satisfying Phi=0) at adjacent source
(i,i+1) and target column n, with target row u≥3 (outside ideal I),
the coefficient coeff(i,i+1; u,n) is forced to zero by pure half-Leibniz
algebra — no I_filtered, kerΦ, or centering required.

The proof splits into five geometric position classes:

| Class | Condition | Equations | Char | Mechanism |
|-------|-----------|-----------|------|-----------|
| u=i | 3≤i≤n-2 | 3 | ≠2 | BoundaryRigidity: three sources provide g=2a, g=2b, g=2b→4a=2a→a=0 |
| u=i+1 | i≥2 | 1 | none | Source=(1,i+1), ptr=(i,i+1), tgt=(1,n). Only T3 fires → direct. |
| u>i+1 | i+1<u<n | 2 | ≠3 | Source=(i,u) gives coeff=coeff; source=(i+1,u) gives coeff=-2*coeff → 3X=0 |
| u<i | u≥3, i+1<n | 1 | none | Source=(1,u), ptr=(i,i+1), tgt=(1,n). Only T3 fires → direct. |
| i=n-1 | u<n-1 | 1 | none | Source=(2,u), ptr=(n-1,n), tgt=(2,n). Reduces to v<n coefficient. |

## Key Insight: T3 is the universal accessor

For adjacent source (i,i+1) with target (u,n) where u≠i, the GOAL
coefficient NEVER appears in bracketIdentity of its own source equation.
It appears as T3 in equations where the SOURCE has column u:

```
source=(p,u), partner=(i,i+1), target=(p,n)
  → T3 = coeff(i,i+1; u,n)  when p=p and u<n
```

This is the mechanism behind u=i+1 (p=1), u<i (p=1), and u>i+1 (p=i or p=i+1).

## non-adjacent source reduction

Non-adjacent sources (j>i+1) are reduced to adjacent via width induction
using `coeffOf_source_exists` + strong induction on n-u. The children
either have:
- v<n (handled by `offdiag_zero_v_lt_n`) — cases IA, IIB
- v=n with smaller n-u measure (induction hypothesis) — case IB

## Proof snippets

All lemmas follow the same structure:
```lean
theorem name (coeff ...) (bounds) (hPhi : ∀ ..., Phi coeff ... = 0) : coeff i (i+1) u n = 0 := by
  have hP := hPhi src_row src_col ptr_row ptr_col tgt_row tgt_col
  unfold Phi at hP
  rw [bracketSource_expansion, bracketIdentity_expansion] at hP
  -- algebraic simplification to reach the conclusion
```

Where bracketSource/bracketIdentity expansions use the standard pattern:
```lean
rw [bracketIdentity_eq_expanded]; split_ifs <;> first | omega | ring
```

## Dependency

ALL five classes depend ONLY on:
- `bracketIdentity_eq_expanded` (expansion formula)
- `bracketSource` definition
- `Phi = 0` (half-Leibniz condition)
- `ring`, `omega` for algebra/arithmetic

NONE depend on: I_filtered, kerΦ, centeredness, coeffOf_cond, Spanning.
