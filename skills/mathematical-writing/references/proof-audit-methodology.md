# Proof Completeness Audit — Systematic Methodology

> Use when verifying that a paper's proof chain is logically complete with no
> gaps, contradictions, or dangling references.

## The Backward-Trace Method

1. **State the final theorem** as a single assertion (e.g., "dim = n+5").
2. **Trace backward**: for each parameter/term in the final tally, identify
   exactly which lemma or computation established it.
3. **Check for dangling references**: claims that say "obtained below" or
   "as shown above" must actually resolve to a specific, earlier statement.
4. **Cross-check the parameter tally** against the reduction claims. If a
   reduction paragraph says "X_k = 0 for all k≥2" but the final tally lists
   X_2 as a survivor, there is an internal contradiction.

## Common Gaps in Derivation-Classification Proofs

### 1. Unsubstantiated coupling claims
Statements like "the chain bridge couples c_k to c_{k+1}" must be backed by
an explicit bracket expansion.  The chain bridge formula
`2π(φ(E_{ij})) = π([φ(E_{ik}),E_{kj}] + [E_{ik},φ(E_{kj})])` only gives a
relation at the non-adjacent source (i,j).  For interior indices, both
bracket terms may vanish, giving `0 = 0` rather than a coupling relation.

**Fix**: For each claimed coupling, write out the chain bridge expansion and
verify that both bracket terms are non-zero at the relevant target.

### 2. Dangling "(obtained below)" references
If a paragraph says "X=0 (obtained below)", verify that the paragraph
"below" actually establishes X=0.  If it doesn't, the proof has a gap —
either X=0 needs to be proved, or the paragraph should not claim it.

### 3. Contradictions between reduction steps and parameter tally
After all reductions, the surviving variables listed in the parameter tally
must be consistent with every reduction claim.  If a reduction says "Y_k = 0
for all k" but the tally lists a Y_k, one of them is wrong.

### 4. Over-claiming propagation
A recurrence `f(k) = g(k) + h(k+1)` does not force `f(k)=0` just because
`h(k+1)=0`.  It gives `f(k) = g(k)`, which may be non-zero.  Propagation
claims must be verified algebraically.

### 5. Boundary index failure in channel restrictions
Index arguments that work for "interior" positions often fail at endpoints.
Example: the claim `[E_{k,k+1}, I] = 0 for k≥3` fails at `k=n-1` because
`[E_{n-1,n}, E_{1,n-1}] = -E_{1,n} ≠ 0`.  For interior k (3≤k≤n-2) the
bracket IS zero.  The endpoint case gives a qualitatively different
equation — often the cross-endpoint constraint itself.

**Fix**: Always verify endpoint cases (k=1, k=n-1) separately from interior
cases.  Do not use "for all k≥3" when the conclusion differs at k=n-1.

## Audit Checklist

- [ ] Every "thus" / "forcing" follows from the immediately preceding
  computation (not from an unstated lemma)
- [ ] Every "by Lemma X" citation uses a lemma whose hypothesis matches
- [ ] Every "by symmetry" argument is backed by explicit involution or map
- [ ] The parameter tally lists exactly the variables that survived all
  reductions, with no extras and no missing ones
- [ ] No paragraph claims a variable is zero while the final tally lists
  it as non-zero (internal contradiction check)
- [ ] All characteristic-dependent steps (char≠2, char≠3) are explicitly
  tied to specific equations
- [ ] Boundary cases (n=4, n=5, k=1, k=n-1) are checked

## Lean Formalization Cross-Reference Audit

When a Lean formalization exists, the paper must be cross-checked against
it.  The Lean proof is **ground truth** — if paper and Lean disagree, the
paper is wrong.

### Method

1. **Locate the Lean normal form lemmas first.**  For derivation-classification
   proofs, these are typically named `*_channel_normal_form` and live in
   `Spanning.lean`.  They declare exactly which positions can be non-zero —
   this is the authoritative answer.  Compare against the paper's surviving
   variable list.

2. **Trace backward from the parameter count** to each reduction step.
   For every "X_k = 0 for k in range R" claim in the paper, find the
   corresponding Lean lemma.  If ranges differ, the paper is imprecise.

3. **Check boundary indices separately.**  Most index arguments have endpoint
   exceptions.  Verify each endpoint (k=1, k=n-1, i=1, j=n) independently.

4. **Verify coupling claims by direct bracket computation.**  Write out the
   chain bridge at the claimed target and compute both bracket terms using
   the bracket formula.  If both evaluate to zero, the coupling is fiction.

### Lean File Map for HalfDer(N_n)

| Lean file | What it proves |
|-----------|---------------|
| `Spanning.lean` | `a_channel_normal_form`, `c_channel_normal_form` — authoritative survivor lists |
| `Centering.lean` | `centered_image_in_I` — image restriction (2266 lines) |
| `BoundaryRigidity.lean` | Case 2(i) of image restriction (3 Ψ equations) |
| `DimensionTheorem.lean` | Final parameter count: n-1 A-generators + 5 boundary cocycles = n+4 |
| `HalfClassification.lean` | `all_diag_equal` — diagonal lemma (complete) |

### Red Flags

- **Dangling references**: "(obtained below)" with no matching proof below
- **Internal contradictions**: reduction says X_k=0, tally says X_k survives
- **Non-existent recurrences**: claimed coupling when both bracket terms = 0
- **Unchecked endpoints**: claims for "k≥3" that fail at k=n-1
