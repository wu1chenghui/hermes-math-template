# Foundation Audit — Primitive Definition Verification

> The final 5% of trust. Verifies that EVERY primitive Lean definition
> matches the paper's mathematical object, not just that the theorems
> correspond.

## When to run

After the Paper ↔ Lean Proof-Level Consistency Audit passes and the paper
is in pre-submission polishing. This is the last verification before
declaring the paper submission-ready.

## Method

For each primitive definition (not theorems — the building blocks):

1. Read the Lean definition and its docstring
2. Compare against the paper's corresponding definition
3. Verify they encode the same mathematical object character-by-character
4. Record the result

## Checklist

Typical primitives to verify (~10-15 items):

| # | Lean | Paper | What to check |
|---|------|-------|---------------|
| 1 | `MatIdx` / `bracket` | §2.1 N_n, bracket formula | Indices 1≤i<j≤n, bracket δ pattern |
| 2 | `HalfDerivation` structure | §2.2 1/2-derivation | The `half_leibniz` field = Ψ=0 |
| 3 | `bracketSource`/`Left`/`Right` | §4.3 BS/BL/BR decomposition | Each component matches Ψ expansion |
| 4 | `Phi = 2·BS − BI` | §2.2 Ψ operator | Exact formula match |
| 5 | `I_target_set` | §2.1 I | {(1,n-1),(1,n),(2,n)} |
| 6 | `I_filtered` | §4.4 Im⊆I | nonzero ⇒ target∈I |
| 7 | `kerPhi` | §4.4 kerΦ | I_filtered ∧ ValidSupp ∧ Φ=0 |
| 8 | `halfDeriv_Id` | §3 e_0 | coeff(i,j,i,j)=1, all others 0 |
| 9 | `scalarCoeff` | §3/§4 | Diagonal coefficient, centered=0 |
| 10 | `coeffOf` | §3 | Valid-index projection with zero defaults |

## Verification standard

For each primitive, open the Lean source and the paper simultaneously.
The definitions must agree **character-by-character** — not just "conceptually
similar."

### Example: MatIdx bracket verification

```lean
-- Lean (MatrixBasis.lean:59-68):
def bracket {n : ℕ} (x y : MatIdx n) : Option (ℤ × MatIdx n) :=
  if h : x.j = y.i then
    some (1, ⟨x.i, y.j, ...⟩)
  else if h' : y.j = x.i then
    some ((-1 : ℤ), ⟨y.i, x.j, ...⟩)
  else none
```

Paper (§2.1): `[E_ij, E_kl] = δ_{jk}E_{il} − δ_{li}E_{kj}`

Verification:
- `x.j = y.i` (j=k) → `(1, E_{i,l})` = `δ_{jk}E_{il}` ✓
- `y.j = x.i` (l=i) → `(-1, E_{k,j})` = `−δ_{li}E_{kj}` ✓
- else `none` = commuting ✓

### Example: kerPhi definition verification

```lean
-- Lean (RepresentationBridge.lean:84-89):
def IsKerPhi (hn : 3 ≤ n) (c : V F) : Prop :=
  I_filtered c hn ∧
  IsValidSupp (n := n) c ∧
  (∀ i j p q u v, ... → Phi c i j p q u v = 0)
```

Paper (§4.4): kerΦ = {f | Im(f) ⊆ I, valid support, Φ(f) = 0}

Verification:
- `I_filtered` = Im(f) ⊆ I ✓
- `IsValidSupp` = valid support ✓
- `Phi = 0` = Φ(f) = 0 ✓

## Anti-patterns

- **DON'T** skip primitives. Foundation errors are the hardest to detect
  later because all theorems build on them.
- **DON'T** accept "conceptually the same." The Lean definition of `bracket`
  returning `Option (ℤ × MatIdx n)` is different from the paper's `δ_{jk}E_{il}`,
  but they encode the same bracket. Verify the encoding is faithful.
- **DON'T** assume Lean's `I_target_set` equals the paper's I without
  checking the exact set of pairs. A typo like `(1, n-2)` instead of
  `(1, n-1)` would propagate through the entire proof.

## Result

All primitives pass → the 5% uncertainty is eliminated. Lean proves
EXACTLY what the paper claims, from the ground up.
