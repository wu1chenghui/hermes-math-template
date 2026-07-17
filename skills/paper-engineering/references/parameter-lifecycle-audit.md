# Parameter Lifecycle Audit

> Trace every variable through Lean from introduction to final dimension count.
> Catches exposition errors where the paper claims a variable is eliminated
> but Lean keeps it alive (or vice versa).

## When to use

- After the paper's proof sketch is stable and Lean-verified
- When a counting claim seems suspicious ("self-pair eliminates n-1 DOFs")
- When multiple reviewers flag the same contradiction between sections
- As part of the Paper↔Lean proof-level consistency audit

## Method

For each variable in the final parameter count (e.g., a_1, a_2, ..., a_{n-1},
b_1, ..., b_{n-1}, c_1, c_2, ..., c_{n-1}), answer:

1. **Which Lean lemma first introduces it?** (Usually a coefficient unfolding)
2. **Which lemmas constrain it?** (F1_coeff_relation, F2_coeff_relation, etc.)
3. **Which lemma (if any) proves it equals zero?** Check ranges carefully.
4. **If it survives, where is it counted in `finrank_halfDer`?**
5. **If it disappears without being killed, identify exactly where.**
6. **Produce the final table of surviving free parameters.**

## Key pitfall: self-pair vacuity

A common error is claiming that self-pair half-Leibniz constraints eliminate
variables. For centered T_0 at adjacent source (k,k+1):

```
Self-pair: [T_0(E_k), E_k] + [E_k, T_0(E_k)] = 0
```

For the c_1 variable (c-channel at k=1):
- BL term: -c_1·[E_{2,n}, E_{12}] = -c_1·(-E_{1,n}) = +c_1·E_{1,n}
- BR term: -c_1·[E_{12}, E_{2,n}] = -c_1·E_{1,n}
- Sum: 0 ← VACUOUS

**Always check both BL and BR terms.** They often cancel. Self-pair is
vacuous for centered derivations whose image lies in I.

## Case study: c_1 lifecycle in N_n classification

The paper originally claimed "self-pair forces c_1 = 0." Lean verification
showed:

- `F2_coeff_relation` (Spanning.lean): constrains c_p for p ≥ 3 only.
  c_1 is NOT in F2's range.
- Self-pair at k=1: BL and BR cancel → vacuous.
- `kerPhi` (RepresentationBridge.lean): target (2,n) ∈ I → c_1 is free.

**Conclusion**: c_1 survives. It is the parameter for basis vector B.
The paper's claim was wrong; the parameter count was accidentally correct
because c_1 was already implicitly counted as one of the 5 boundary cocycles.

## Verification checklist

- [ ] For each "eliminated" claim in the paper, verify the Lean lemma's index range covers it
- [ ] For each "surviving" variable, confirm it appears in Lean's basis/spanning set
- [ ] Check all self-pair claims: compute BL + BR, don't assume non-cancellation
- [ ] Verify the raw parameter count minus relations equals Lean's `finrank`
- [ ] Cross-reference the paper's explicit basis (§3) with Lean's `Gset`/spanning set
