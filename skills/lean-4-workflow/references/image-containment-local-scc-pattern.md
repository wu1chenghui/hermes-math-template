# ImageContainment: Local SCC over Width-Chain Pattern

> Discovered 2026-06-27 during the D3 ImageContainment dependency audit.

## The Pattern

The original blueprint designed `width_c_chain_zero` and `width_a_chain_zero` as
descending-induction chain proofs to close the c-channel (target (2,n)) and a-channel
(target (1,n-1)) coefficients.  Both lemmas turned out to be **orphaned** — the actual
dispatch uses local 3×3 SCCs instead.

### Local SCCs that replaced chain machinery

| Lemma | Mechanism | det | char | What it replaced |
|-------|-----------|-----|------|-----------------|
| `boundary_local_234` | 3×3 SCC, commuting partners | 1 | ≠2 | BND-L dispatch (was: width_c_chain) |
| `boundary_local_right` | 3×3 SCC, commuting partners | 1 | ≠2 | BND-R u=3 (was: width_a_chain) |
| `boundary_coupling` | 2×2 coupling, det=3 | 3 | ≠3 | Gate-5 cycle node |
| `boundary_coupling_sigma` | σ-mirror of above | 3 | ≠3 | D-fires in width_a |

### The σ-mirror construction pattern

When proving a RIGHT-boundary coefficient `coeff(n-1,n; u, v)`:
1. Take the LEFT-boundary lemma `coeff(1,2; u', v')`
2. Apply the anti-automorphism: `(i,j) → (n+1-j, n+1-i)`
3. Verify bracketSource=0 for the mirrored partners
4. Check that the det is invariant under mirroring

BUT: not all mirrors are literal.  `boundary_local_right` is NOT the literal σ-mirror
of `boundary_local_234` — it uses different partners (1,3) and (2,3) vs (3,n) and (3,4).
The pattern is "same 3×3 structure, different indices."

## Dependency Audit Methodology

When facing multiple sorries in a proof block:

1. **List all sorries** with exact coefficient signatures
2. **Check caller graph**: `search_files` for lemma name across the project — if zero
   callers, the lemma is orphaned and its sorries are not live blockers
3. **Classify each sorry**: live / deferred-to-later-phase / orphaned
4. **Build minimal dependency graph** of LIVE sorries only
5. **Identify SCCs**: which live sorries form irreducible clusters?

### Orphaned lemma annotation convention

```lean
/-- **(CURRENTLY UNUSED — YYYY-MM-DD)** <description>.
    This lemma has no callers in the current pipeline: <reason>.
    Retained as a library lemma / alternative argument.
    The remaining `sorry` slots are NOT blockers.
    ... -/
```

Do NOT delete orphaned lemmas — they may be needed by RepresentationBridge,
Spanning, DimensionTheorem, or future library consumers.

## Session-specific discoveries (2026-06-27)

### width_c / width_a are orphaned
- `width_c_chain_zero`: zero callers. BND-L now uses `boundary_family` (local SCC).
- `width_a_chain_zero`: zero callers. D-fires uses `boundary_coupling_sigma`.
- `width_stability_c_regrounded_i1`: zero callers. GATE-B prototype only.

### internal_det_coupling u≥3 chain mechanism
```
bracketIdentity(i,i+1; u-1,u; u-1,i+1):
  coeff(i,i+1; u, i+1) + coeff(u-1,u; u-1, i) = 0

Descending chain (u-1 steps):
  coeff(u-1,u; u-1,i) + coeff(u-2,u-1; u-2,i) = 0
  ...
  coeff(2,3; 2,i) + coeff(1,2; 1,i) = 0

Terminal: coeff(1,2; 1,i) = 0 via boundary_family (requires 3≤i≤n-2)
```
This is a finite chain (not infinite induction), implementable via `Nat.decreasingInduction`.
Edge case i=n-1: terminal ∈ I, needs different handling.
Edge case u=1: no commuting partner exists (partner (0,1) invalid).

### Live blocker count after audit
Started: 6 errors + 14 sorries → After fixes: 0 errors + 13 sorries
Of the 13: 4 are in orphaned lemmas (not blockers), 4 deferred to §D centering,
leaving ~5 genuinely live sorries in the ImageContainment pipeline.
