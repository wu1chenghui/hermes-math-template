# Constraint Provenance Methodology

> Derived from the HalfDer paper refinement (2026-07-02).
> When a paper claims "mechanism X eliminates N degrees of freedom,"
> verify with computation before trusting the claim.

## When to Use

The paper claims a specific constraint mechanism (e.g., "self-pair
eliminates n-1 degrees of freedom") as part of a parameter counting
argument. Before accepting this claim, verify it with a full
constraint matrix computation for a small fixed n.

## Method

1. **Fix a small n** (e.g., n=5). List all sources, all targets,
   all channel variables.

2. **Build the full constraint matrix** over ℚ (exact rational
   arithmetic). Include ALL half-Leibniz equations for all basis
   pairs, not just the ones the paper highlights.

3. **Compute rank with and without each constraint class**:
   - HL only (no image restriction)
   - HL + Image restriction
   - HL + Image + Self-pair
   - HL + Image + Self-pair + F_eq

4. **Compute marginal rank contribution** of each class. The
   class with the largest marginal contribution is the dominant
   mechanism.

5. **Compare with paper's narrative**. If the paper claims class X
   is the main mechanism but computation shows class Y contributes
   more rank, the paper's narrative is wrong.

## HalfDer Case Study (n=5)

| Constraint class | Cumulative rank | Marginal |
|-----------------|:---:|:---:|
| HL only | 40 | 40 |
| + Image restriction | 90 | **50** |
| + Self-pair | ~90 | ~1 |
| + F_eq | ~90 | ~1 |

**Finding**: Image restriction is the dominant constraint (~55% of
total rank). Self-pair contributes at most 1 constraint (c₁=0),
not n-1 as the paper originally claimed.

## Python Verification Script

```python
from fractions import Fraction as F

# Build sources for N_n
sources = [(i,j) for i in range(1,n) for j in range(i+1,n+1)]
d = len(sources)

# Build bracket function
def bracket(a, b):
    i,j = a; k,l = b
    # ... returns (coeff, result_index) or (0, None)

# Build constraint matrix
for si in range(d):
    for pi in range(si, d):
        for ti in range(d):
            row = [F(0,1)] * (d*d)
            # BS: 2*T[bracket] at target
            # BL: -[T(src), partner] at target
            # BR: -[src, T(partner)] at target
            # ...

# Gaussian elimination over ℚ
# Measure rank with each constraint subset
```

## Key Insight

The paper's narrative and the actual mathematical mechanism can
diverge. The paper might tell a clean story ("self-pair eliminates
n-1 DOFs") while the computation shows a different story ("image
restriction is dominant, self-pair is almost vacuous"). Always
verify with computation before finalizing the paper's explanation.
