# Consistency Audit: Check Theorem Statements Before Engineering

**When**: before filling sorries that touch diagonal/centering/boundary
coefficients, or when a proof branch seems "harder than it should be".

**Why**: A theorem stated for `∀ D : HalfDerivation F n` may be FALSE for the
identity derivation, which has `coeff(i,j;i,j) = 1` for all valid (i,j).
If the identity is a counterexample, the statement needs a centering hypothesis.

## The pattern

1. **Identify a concrete test object** — usually the identity derivation `Id`.
2. **Compute one concrete value** — `coeffOf(Id, i, j, u, v)` for a case where
   the theorem claims 0.
3. **If Id gives non-zero**: the theorem statement is false as written. It needs
   a centering hypothesis: `D.coeff ∈ kerΦ hn` or `∀ i j, D.coeff i j i j = 0`.
4. **If Id gives 0**: the statement survives this test (but may fail for other
   test objects — try Type B/C/D/E/F basis vectors as additional probes).

## Worked example (this project, 2026-06-26)

`imageContainment_skeleton` and `width_c_chain_zero` were stated for arbitrary
`HalfDerivation F n`. For `Id`:
- `coeffOf(Id, 2, n, 2, n) = 1` → violates `width_c_chain_zero`
- `coeffOf(Id, 3, n, 3, n) = 1` → violates `imageContainment_skeleton`

**Resolution**: the blueprint (lines 24-25, 49-50) and paper (line 99) specify
that image containment targets the **centered component** or **off-diagonal
targets**. The Lean skeleton was missing this qualifier.

**Fix applied**: `imageContainment_skeleton` gets `(h_off_diag : i ≠ u ∨ j ≠ v)`
— the off-diagonal qualifier. This keeps the primitive theorem minimal.
D3.4 will build the **centered wrapper** on top:

```
imageContainment_offdiag    ← primitive (h_off_diag, true for all D)
        │
        └── + all_diag_equal + trace → all diagonal = 0
              │
              ▼
        imageContainment_centered    ← wrapper (D3.4, called on E = D - λ·Id)
```

**Key design principle**: keep primitive theorems as weak as possible (off-diag
qualifier), build structural hypotheses (centering) into wrappers. This avoids
embedding a hypothesis that the theorem itself is trying to prove.

**Zero callers audit**: before modifying a theorem signature, check who calls it.
`imageContainment_skeleton` had zero callers — safe to change. If callers exist,
modify the call sites or create a wrapper instead.

## Related: hI-free vs hI-based lemma layering

When a lemma like `width_stability_c` takes `hI : I_filtered D.coeff hn` (the
full theorem-level condition), it CANNOT be called from within a proof that is
trying to establish `hI`. The call graph must be a DAG:

```
CORRECT:                           WRONG (cycle):
  Spanning (hI known)                imageContainment (proving hI)
    │                                    │
    └── width_stability_c (uses hI)      └── width_c_chain_zero (hI-free)
                                               │
                                               └── needs hI → cycle
```

If a lemma needs a hypothesis that isn't available at its call site, either:
- Create a weaker hI-free version (but verify it doesn't create cycles)
- Restructure the call graph so the hypothesis IS available
- Accept that the lemma belongs in a different proof layer

In this project, `width_c_chain_zero` (hI-free) was created to avoid the hI
dependency, but it introduced a structural cycle via C-fire → b-channel →
width_c_chain. The underlying issue: `width_stability_c` (hI-based) was designed
for the Spanning layer, not for imageContainment. Moving it to imageContainment
without hI created a proof that doesn't close.

## Checklist

- [ ] Pick a concrete test object (Id, Type B, Type C, ...)
- [ ] Compute `coeffOf(test, i, j, u, v)` for one ∉I target
- [ ] If non-zero: the theorem needs a centering hypothesis
- [ ] Verify existing compiled lemmas are not invalidated by the added hypothesis
- [ ] Update theorem signatures BEFORE filling sorries
