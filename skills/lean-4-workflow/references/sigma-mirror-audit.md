# σ-Mirror Audit Methodology

> When mirroring a Lean proof under the N_n symmetry σ: E_{i,j} → E_{n+1-j, n+1-i},
> do NOT mechanically copy the proof. Run this audit first.

## Step 1: Map the coeffOf_cond survivors

For a `coeffOf_cond` split at k = i+1 (descending induction on source row):

```
A: if u<k ∧ v=j' then coeff(i,k; u,k)       [adjacent interior]
B: if u=k ∧ j'<v then coeff(i,k; j',v)       [boundary leaf]
C: if u=i ∧ k<v then coeff(k,j'; k,v)        [wider source]
D: if u<i ∧ v=k then coeff(k,j'; u,i)        [reverse boundary]
```

The σ-mirror changes the target (u,v) → σ(u,v). Recompute which of A/B/C/D fire and at which k-values. **Do not assume the same cases fire.**

### Example: width_c → width_a

| Term | width_c (target 2,n) | width_a (target 1,n-1) |
|------|----------------------|------------------------|
| A | k≥2, j'=n | k≥1, j'=n-1 |
| B | k=1 | NEVER (k=0 impossible) |
| C | k=2 | k=1 |
| D | never (n≠k+1) | k=n-2 |

**Key finding**: B disappears in the mirror, C shifts from k=2 to k=1, D appears at k=n-2.

## Step 2: Check commuting partners

For each bracketIdentity used in the original proof, verify the σ-mirror partners still commute under the new indices. Common pitfall: the original partner commutes because of specific index inequalities that may not hold after σ.

## Step 3: Check for branch disappearance

After σ, some branches may become:
- **Unreachable** (like B in width_a: k=0 impossible) → `exfalso` with omega
- **Diagonal** (target = source) → excluded by h_off_diag in the induction context
- **∈I target** (target lands in the ideal I) → may need different machinery

## Step 4: Identify which lemma handles each survivor

Map each surviving child coefficient to its closure lemma:
- Adjacent interior coeff(k,k+1; u,k+1): `internal_det_step`, `a_fire_scc_close`, `internal_det_step_left`
- Boundary leaf: `boundary_family`, `boundary_family_right`
- Wider source: `width_c_chain_zero`, `width_a_chain_zero` (induction hypothesis)
- ∈I target residual: `width_stability_a/c` or centering machinery

## Case study: `boundary_coupling` → `boundary_coupling_sigma`

The cleanest σ-mirror in the project.  Original at source (1,2), targets (3,n) and (1,3;2,n).
Under σ: source (n-1,n), targets (1,n-2) and (n-2,n;1,n-1).

### Original mechanism (char≠3, det=3)
```
R_a: bracketIdentity(1,2; 1,3; 1,n) = 0  →  x = y   [x=coeff(1,2;3,n), y=coeff(1,3;2,n)]
    T2: 1=1 ∧ 3<n → coeff(1,2;3,n)   T3: 1=1 ∧ 2<n → coeff(1,3;2,n)
R_b: coeffOf_cond(1,3;2,n) @ k=2       →  2y = -x  [B fires: 2=2 ∧ 3<n]
→ 2y = -y → 3y = 0 → y = 0 → x = 0
```

### Mirror mechanism (identically det=3, char≠3)
```
R_a': bracketIdentity(n-1,n; n-2,n; 1,n) = 0  →  x' = y'  [x'=coeff(n-1,n;1,n-2), y'=coeff(n-2,n;1,n-1)]
    T1: 1<n-2 ∧ n=n → coeff(n-1,n;1,n-2)   T4: 1<n-1 ∧ n=n → coeff(n-2,n;1,n-1)
R_b': coeffOf_cond(n-2,n;1,n-1) @ k=n-1     →  2y' = -x'  [D fires: 1<n-2 ∧ n-1=n-1]
→ 2y' = -y' → 3y' = 0 → y' = 0 → x' = 0
```

Key observations:
- **Partner changes**: (1,3) → (n-2,n).  Crucially, the partner in the mirror is (n-2,n)
  NOT (n-2,n-1) — the bracketIdentity T1/T4 pattern differs from the T2/T3 pattern
  in the original.  Always recompute which T's fire.
- **coeffOf_cond term changes**: B fires in original → D fires in mirror.  The split
  point shifts from k=2 to k=n-1.
- **bracketSource stays zero**: Both originals use commuting partners, and the mirrors
  do too (n≠n-2, n-1≠n).
- **det unchanged**: det=3 in both, char≠3 requirement preserved.

### D fires in width_a uses this

D fires 1 (width-2 case, k=n-2): `coeffOf_cond @ k+1=n-1` → D fires → child =
`coeff(n-1,n;1,n-2)`.  Closed by `boundary_coupling_sigma.1`.

D fires 2 (wider case): requires j' > n AND j' ≤ n → **UNREACHABLE** (`exfalso; omega`).

> **σ preserves the internal_det structure, but does NOT preserve the width-chain case decomposition.**

This means:
- `internal_det_step` → `internal_det_step_a`: clean mirror
- `width_c_chain_zero` → `width_a_chain_zero`: case distribution changes, audit required

> **Not every "mirror" is a literal σ-map (2026-06-27).**  `boundary_local_right`
> (right-boundary 3×3 SCC for u=3) is NOT the σ-image of `boundary_local_234`.
> The σ-map of boundary_local_234 gives coeff(n-1,n;n-2,n)=0 etc., which is a
> different 3×3 system.  `boundary_local_right` uses DIFFERENT partner indices
> — (2,3) and (1,3) — chosen to isolate the GOAL coefficient coeff(n-1,n;3,n).
> **Lesson**: when constructing a mirror, (1) try the literal σ-map first,
> (2) if the indices don't match the goal, derive a new self-contained SCC
> using the SAME proof structure (2 bracketIdentities + 1 coeffOf_cond) but
> with recomputed partner indices that isolate YOUR target coefficient.
> The proof structure transfers; the indices may not.

## Common Lean pitfalls in mirror proofs

1. **`subst` eliminates variables**: After `subst hi_n`, the variable `n` is replaced by `i+1` everywhere. You can no longer reference `n`.
2. **`bracket_zero` argument count**: Takes 11 proofs after 6 indices: hi, hij, hjn, hc, hcd, hdn, hu, huv, hvn, hj_ne_c, hi_ne_d. Count carefully — omega can hide an off-by-one.
3. **`rw` vs `rw at`**: Top-level `rw [h_eq]` only rewrites the GOAL, not hypotheses. Use `rw [h_eq] at h_notI huv` to rewrite hypotheses.
4. **`bracketIdentity` signs**: The expansion is T1 - T2 + T3 - T4. T2 and T4 have minus signs. When only T2/T4 fire, the result may be `-coeff = 0` rather than `coeff = 0`.
