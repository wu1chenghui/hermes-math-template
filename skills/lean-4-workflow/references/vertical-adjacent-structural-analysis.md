# Vertical Adjacent Coefficient Structural Analysis

When a coefficient resists closure through normal induction, first EXHAUSTIVELY
map its bracketIdentity coupling structure BEFORE attempting new mechanisms.

## The method (delta-probe enumeration)

1. **Define a delta coefficient**: nonzero only at the target coefficient.
2. **Enumerate ALL valid bracketIdentity calls** (full 6D loop over a,b,c,d,u,v).
3. **Record which calls produce non-zero** with the delta — these are the ONLY
   bracketIdentity equations that involve the coefficient.
4. **Classify the coupling**: for each hit, identify whether the residual lands on
   I-targeted, ∉I-smaller-target-width, or ∉I-larger-target-width coefficients.

## Worked example: V(t) = coeff(t, t+1; t, n)

For t=4, n=10, full enumeration gave:

```
T2 family (source-side extraction, minus sign):
  bracketIdentity(t, t+1, c, t, c, n) = -V(t)   for ALL c < t

T3 family (partner-side extraction, plus sign):
  bracketIdentity(a, t, t, t+1, a, n) = +V(t)   for ALL a < t
```

**Zero other hits.** This means V(t) couples ONLY through these two families.

Both families produce the same half-Leibniz equation:
  V(t) = 2·coeff(r, t+1; r, n)   for any r < t.

Choosing r=1 gives V(t) = 2·coeff(1, t+1; 1, n) — coupling to I-targeted
b-channel. Choosing r≥3 gives coupling to ∉I wider-target coefficients
(C-fire chain).

## Structural theorem

V(t) NEVER appears in bracketSource (proof: for source (t,t+1) with target
(t,n) to be the output of a Lie bracket, we need either j=c with (i,d)=(t,t+1)
requiring t<j<t+1 impossible, or i=d with (c,j)=(t,t+1) requiring t+1<i<t+1
impossible). Therefore half-Leibniz for V(t) is always `0 = bracketIdentity`,
never `2V = bracketIdentity`. This is WHY there is no SCC — V(t) is a leaf,
not a cycle node.

## Coupling chain

```
V(t) = coeff(t, t+1; t, n)
    │  half-Leibniz (T2/T3 family, r=1 chosen)
    ▼
  2·coeff(1, t+1; 1, n)       [b-channel, I-targeted]
    │  width_b_zero (Term C, split at 2)
    ▼
  coeff(2, t+1; 2, n)          [c-channel]
    │  = width_stability_c → width_c_chain_zero
    ▼
  width_c_chain_zero closure
```

## What this established

- V(t) is a PURE CONSUMER of `width_c_chain_zero`. No new mechanism needed.
- Q-invariant, vertical SCC, width_stability_b — all WRONG approaches.
- The coupling to diagonal smaller-target-width terms was a FALSE HOPE
  (corrected by full enumeration — the T3 condition u=1≠3 blocked it).

## The rule

> Exhaustive delta-probe BEFORE architecture freeze.
> If the probe says only two families, do NOT invent a third mechanism.
