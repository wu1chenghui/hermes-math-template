# BND-L Boundary Dispatch — 3-Equation SCC Closure (hI-free)

## When to use

You are implementing `imageContainment_skeleton`'s BND-L dispatch (source (1,2), target (1,v)
with v≠2, (1,v)∉I), and the bracketIdentity produces a **wide ∈I residual** like
`coeff(3,n;2,n)` or `coeff(3,v;2,v)`. The blueprint says "kill with `width_stability_c`"
— but `width_stability_c` needs `hI` (I_filtered), which `imageContainment` does not
have (it's proving I_filtered). This reference provides an **hI-free, self-contained**
closure using a 3-equation algebraic SCC.

## The 3-equation SCC (for v=3, the base case)

Target: `coeff(1,2;1,3) = 0`. Three unknowns form a closed system:

```
  x = coeff(3,n;2,n)     (wide ∈I residual, from bracketIdentity T3)
  y = coeff(3,4;2,4)     (adjacent off-diag ∉I, from coeffOf_cond)
  z = coeff(1,2;1,3)     (boundary target, from bracketIdentity T1)
```

### Equation 1: bracketIdentity(1,2, 3,n, 1,n) = 0

Partner (3,n) commutes with (1,2): [E_{1,2}, E_{3,n}] = 0. bracketSource = 0.
bracketIdentity expansion (target (1,n)):
  T1: u<c? 1<3 ✓, v=d? n=n ✓ → +coeff(1,2;1,3)
  T4: u<a? 1<1 ✗
  → z + x = 0    ... (1)

### Equation 2: bracketIdentity(3,4, 1,2, 1,4) = 0

Partner (1,2) commutes with (3,4): 4≠1, 3≠2. bracketSource = 0.
bracketIdentity expansion (target (1,4)):
  T2: u=c? 1=1 ✓, d<v? 2<4 ✓ → -coeff(3,4;2,4)
  T4: u<a? 1<3 ✓, v=b? 4=4 ✓ → -coeff(1,2;1,3)
  → -(y + z) = 0  → y + z = 0    ... (2)

### Equation 3: coeffOf_cond(3,n;2,n) at k=4

coeffOf_cond_of D 3 n 2 n ... k=4:
  A: u<k? 2<4 ✓, v=j? n=n ✓ → coeffOf(3,4;2,4)   [sole survivor]
  B: u=k? 2=4 ✗
  C: u=i? 2=3 ✗
  D: u<i? 2<3 ✓, v=k? n=4 ✗
→ 2x = y    ... (3)

### Closure

(3): y = 2x. (1): z = -x. (2): y = -z = x.
→ 2x = x → x = 0 → y = z = 0.
**det=1, char-independent** (only needs 2≠1 in F).

## Extension to general v>3 (CORRECTED 2026-06-26g)

The original analysis claimed bracketIdentity(1,2, 3,v, 1,v) gives
`coeff(1,2;1,v) + coeff(3,v;2,v) = 0`. This is WRONG — the T1 term is
`coeff(1,2;1,3)` (depends on c=3, not on v), NOT `coeff(1,2;1,v)`.

The correct approach uses bracketIdentity with partner (v, v+1):

  bracketIdentity(1,2, v, v+1, 1, v+1) = 0  [commuting: 2≠v (v≥3), 1≠v+1]
  → coeff(1,2;1,v) + coeff(v,v+1;2,v+1) = 0

The interior coeff(v,v+1;2,v+1) is an ADJACENT source with target (2,v+1) ∉ I
(for v ≤ n-2, v+1 ≠ n), killed by `internal_det_step`.

This gives the `boundary_family` lemma in ImageContainment.lean:

  boundary_family D hn hn5 h3 v hv3 hvn2 : coeffOf D 1 2 1 v = 0

Covers all v with 3 ≤ v ≤ n-2, i.e. all (1,v) ∉ I.

## Implemented lemmas (ImageContainment.lean, 2026-06-26g)

| Lemma | Signature | Dependencies |
|---|---|---|
| `boundary_local_234` | `hn5 → coeff(1,2;1,3)=0 ∧ coeff(3,4;2,4)=0 ∧ coeff(3,n;2,n)=0` | bracket_zero×2, coeffOf_cond_of |
| `boundary_residual_shape` | `hv5 → hvn → 2·coeff(3,v;2,v) = coeff(3,4;2,4)` | coeffOf_cond_of only |
| `boundary_family` | `hn,hn5,h3,v,hv3,hvn2 → coeff(1,2;1,v)=0` | boundary_local_234 + internal_det_step |
| `boundary_rep_A` (rewritten) | `(_hn)(_hI) hn5 → coeff(1,2;1,3)=0` | → boundary_local_234.1 |

All are hI-free, induction-free, width-machinery-free.

## Key structural insight

The wide ∈I residual `coeff(3,v;2,v)` reduces via coeffOf_cond at k=4 to
**the same adjacent child** `coeff(3,4;2,4)` for ALL v≥5. This child is a
single coefficient that is closed once (in the v=3 base case) and reused for
all other boundary targets.

## Comparison with blueprint

Blueprint line 226 says "→ width_c" (call `width_stability_c`). This requires hI.
The 3-equation SCC provides an alternative that is:
- hI-free (no I_filtered needed)
- self-contained (only bracketIdentity + coeffOf_cond)
- char-independent (det=1)
- Does NOT require `width_c_chain_zero` or `width_stability_c`

## BND-R σ-mirror

The BND-R dispatch (source (n-1,n), target (u,n)) has the symmetric closure
via width_a counterparts:

  x' = coeff(1,3;1,n-1)    (wide ∈I, a-channel)
  y' = coeff(n-1,n-2;1,n-2) (adjacent off-diag)
  z' = coeff(n-1,n;1,n-1)   (boundary target)

Equations 1-3 are the σ-mirror: swap (1,2)↔(n-1,n), (2,n)↔(1,n-1),
k=4 ↔ k=n-3 in coeffOf_cond.

## Implementation notes

- Use `if_pos`/`if_neg` for ∧-conditions (NOT `simp` which may unfold the ∧).
- Use `bracket_zero` to get bracketIdentity=0 (commuting partners).
- Use `coeffOf_f` to bridge coeffOf ↔ D.coeff when needed.
- For the base case closure, use `congrArg Neg.neg` to negate the half-Leibniz
  equation when needed, then `simp` to simplify.
- The pattern generalises: for ANY wide ∈I residual that reduces via coeffOf_cond
  to a single adjacent child, check whether bracketIdentity with the child's source
  can form a self-contained SCC.

### Pitfalls encountered during implementation

1. **`bracket_zero` argument count**: exactly **11** proof arguments after the 6
   indices: hi, hij, hjn (3) + hc, hcd, hdn (3) + hu, huv, hvn (3) + hj_ne_c,
   hi_ne_d (2) = 11. Using 10 or 12 triggers confusing type errors. All can be
   `(by omega)` when the indices are concrete numbers and n≥5.

2. **`bracketIdentity_eq_expanded` sign**: T2 (`u=c ∧ d<v`) produces `- coeff(i,j;d,v)`
   and T4 (`u<i ∧ v=j`) produces `- coeff(c,d;u,i)`. After `bracket_zero`, the
   equation is `±coeff ± coeff ± ... = 0`. Use `simpa [sub_eq_add_neg]` to
   simplify, or `linear_combination` to extract the desired relation.

3. **`(1,v) ∉ I_target_set hn` → `v ≤ n-2`**: don't `simp` the membership (it
   produces `True ∧ ... ∨ ... → False`). Use the direct contradiction pattern:
   ```lean
   have hv_ne_n : v ≠ n := by intro heq; apply h_notI; simp [I_target_set, heq]
   have hv_ne_n1 : v ≠ n - 1 := by intro heq; apply h_notI; simp [I_target_set, heq]
   omega
   ```

4. **Family extension with bracketIdentity**: the natural partner (3,v) gives
   `coeff(1,2;1,3) + coeff(3,v;2,v) = 0` — the T1 term is fixed at `(1,3)`,
   not the target `(1,v)`. To get `coeff(1,2;1,v)` use partner (v, v+1) instead.
