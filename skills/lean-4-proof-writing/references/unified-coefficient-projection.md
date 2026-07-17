# Unified Coefficient Projection — When N Lemmas Are Actually One

This pattern was discovered during the June 2026 attack on the final remaining
gap of the 1/2-derivation classification project (N_n, dim = n). It generalizes
the Semantics Bridge pattern to a deeper structural insight.

## The signal

You have N sub-lemmas that all look like:

```
± coeffOf D src1 idx1 idx2 ± coeffOf D src2 idx3 idx4 = 0
```

They have names like `pair_cancel_AC`, `pair_cancel_AD`, `only_A`, `only_B` ...
and each seems to require its own proof strategy (embedding, typeA, typeB,
boundary cases).

**Danger signal**: every time you decompose, you discover new boundary cases.
The difficulty is not decreasing — it's being redistributed.

## The insight

These N lemmas are NOT N independent mathematical propositions. They are
**N coordinate projections of ONE operator-level identity**.

### Derivation

Let A = E(i,i+1) (adjacent source), B = E(c,d) (any commuting source).

Expand D in basis: D(A) = Σ a_{pq}·E_{pq}, D(B) = Σ b_{pq}·E_{pq}.

The operator identity [D(A), B] + [A, D(B)] = 0 (when [A,B]=0),
projected onto basis element E(u,v), yields the **unified formula**:

```
δ(v,d)·a_{u,c} − δ(u,c)·a_{d,v} + δ(u,i)·b_{i+1,v} − δ(v,i+1)·b_{u,i} = 0   (★)
```

Where:
- a_{u,c} = coeffOf D i (i+1) u c
- b_{i+1,v} = coeffOf D c d (i+1) v
- etc.

### Specializing (★) to each lemma

Every pair/only lemma is just (★) with specific (u,v) and inequality
assumptions that force three of the four Kronecker δ's to zero:

| Lemma | (u,v) | δ's that survive | Reduces to |
|-------|-------|------------------|------------|
| pair_cancel_AC | (i,d) | δ(v,d)=1, δ(u,i)=1 | a_{i,c} + b_{i+1,d} = 0 |
| pair_cancel_AD | (u,i+1) | δ(v,d)=1, δ(v,i+1)=1 | a_{u,c} − b_{u,i} = 0 |
| pair_cancel_BC | (c,v) | δ(u,c)=1, δ(u,i)=1 | −a_{d,v} + b_{i+1,v} = 0 |
| pair_cancel_BD | (c,i+1) | δ(u,c)=1, δ(v,i+1)=1 | −a_{d,v} − b_{u,i} = 0 |
| only_A/B/C/D | various | one δ survives | single term = 0 |

## The architectural lesson

**Before** (8 independent proofs):

```
pair_cancel_AC → embedding → typeA → boundary
pair_cancel_AD → embedding → typeA → boundary
pair_cancel_BC → embedding → typeB → boundary
...
```

**After** (1 proof + 8 specializations):

```
unified_formula (★) → proved once (via whatever approach works)
    ├── pair_cancel_AC := rw [unified_formula]; simp; omega
    ├── pair_cancel_AD := rw [unified_formula]; simp; omega
    ├── ...
    └── only_D       := rw [unified_formula]; simp; omega
```

The boundary difficulty does NOT disappear — (★) still needs a proof. But
it only needs to be solved ONCE, not 8 times with 8 different proof strategies.

## Relationship to Semantics Bridge

The Semantics Bridge pattern (see `references/semantics-bridge-pattern.md`)
lifts a 16-case `split_ifs` into a 2-branch `by_cases`. This unified
projection pattern goes one level deeper: it recognizes that even the
sub-lemmas of the adjacent case are NOT independent, but are all
projections of the same operator identity.

```
Semantics Bridge:  16 cases → 1 operator identity (non-adjacent)
Unified Projection: 8 lemmas → 1 operator identity (adjacent)
```

## When to apply this pattern

- You have ≥4 lemmas with similar structure (± coefficient pairs)
- Each lemma's hypotheses include inequality constraints on indices
- The constraints in each lemma force most Kronecker δ's to 0/1
- The remaining terms are just coefficient projections from different sources

## Verification step

Before writing any Lean code for the N sub-lemmas:

1. Derive the operator identity on paper (bracket expansion + coefficient projection)
2. Write the unified formula with Kronecker δ's
3. For each sub-lemma, plug in (u,v) and verify the δ's reduce correctly
4. If all N reduce to the same formula, you have ONE proof target, not N

If step 3 fails for some lemma, the decomposition IS genuine and separate
proofs are warranted. But this should be the exception, not the default.
