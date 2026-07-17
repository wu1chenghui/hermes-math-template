# BND-L Boundary 3-Equation Local SCC

Session: 2026-06-26, checkpoints f and g.
File: `E/Classification/ImageContainment.lean`.

## Pattern

The BND-L boundary dispatch (source (1,2), target (1,v) with (1,v)∉I) can be
closed WITHOUT hI, width machinery, or induction, using two purely local mechanisms:

### Seed: `boundary_local_234` — 3×3 self-contained SCC

Three coefficients x = coeff(3,n;2,n), y = coeff(3,4;2,4), z = coeff(1,2;1,3):

```
(1) bracketIdentity(1,2, 3,n, 1,n) = 0  →  z + x = 0   [commuting: 2≠3, 1≠n]
(2) bracketIdentity(3,4, 1,2, 1,4) = 0  →  -y - z = 0   [commuting: 4≠1, 3≠2]
(3) coeffOf_cond(3,n;2,n) @ k=4        →  2x = y        [sole survivor: A-term]
```

Solve: (1)-(2) → x = y. (3): 2x = x → x=0 → y=z=0.
det=1, char-independent (2x=x → x=0 uses only 1≠0, true in any field).
NO hI, NO induction, NO width_stability_c.

### Reduction: `boundary_residual_shape` — uniform coeffOf_cond

For all v ≥ 5: `coeffOf_cond(3,v;2,v)` at k=4 collapses to:
`2·coeff(3,v;2,v) = coeff(3,4;2,4)`
A-term always fires (2<4, target=v); B/C/D all impossible (2≠4, 2≠3, v≠4).
Uniform for the entire family — the whole right-side of the boundary chain
reduces to the single coefficient y = coeff(3,4;2,4) already killed by the SCC.

### Family: `boundary_family` — bracketIdentity + interior det

For v ≥ 4: bracketIdentity(1,2, v,v+1, 1,v+1) → coeff(1,2;1,v) + coeff(v,v+1;2,v+1) = 0.
Interior coeff(v,v+1;2,v+1) closed by `internal_det_step`.
v=3 edge case: direct from `boundary_local_234.1`.

Covers ALL v with 3 ≤ v ≤ n-2 (full family coeff(1,2;1,v)=0).
Total dependency: only bracket_zero + coeffOf_cond_of + internal_det_step.
Zero hI. Zero width machinery.

## Key technique

**Commuting-partner bracketIdentity** is the universal primitive. For any pair of
sources (i,j) and (c,d) that commute (j≠c and i≠d), `D.bracket_zero` gives
`bracketIdentity = 0`, which expands to a linear relation between at most 4
coefficient terms. Picking the right partner isolates the goal coefficient.

## Engineering discipline (user preference)

- Keep OLD proof archived (e.g. `_old` suffix) before replacing. Delete only after
  the new proof is verified and all callers compile.
- Verify with `lean_diagnostic_messages(severity='error')` after each step.
- Do NOT restructure dispatch slots unless proven unreachable.
- BND-R dispatch at `u=i, i+1=n` in `imageContainment_skeleton` is UNREACHABLE
  (target = source diagonal excluded by h_off_diag). Closed via `exfalso`.
  The real BND-R boundary handling belongs in the `u<i` dispatch branch with
  source (n-1,n) and target row u < n-1.
