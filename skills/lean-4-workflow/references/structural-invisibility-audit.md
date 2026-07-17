# Structural Invisibility Audit

When a coefficient resists closure after partner audit and local SCC
enumeration, it may be **structurally invisible** — meaning it does NOT
appear in any half-Leibniz equation (T1-T4 or bracketSource) for ANY
valid source/partner/target triple.

## When to run this audit

- A coefficient has resisted closure through 3+ rounds of partner enumeration
- You suspect the skeleton comment claiming "bracketSource couples X to Y" may be wrong
- You need to decide: "defer to §D" vs "keep searching for a mechanism"

## Method

### 1. Fix the GOAL coefficient

GOAL = coeff(si, sj; tu, tv), e.g. coeff(i, i+1; i, n).

### 2. Enumerate all T1-T4 positions

For T1-T4 in `bracketIdentity_eq_expanded`:

| Term | Matches GOAL when | Valid? |
|------|-------------------|--------|
| T1 = coeff(si,sj; u, c) | source=(si,sj), target_row=tu, partner_row=c=tv | Check: u < c (tu < tv ✓), d = tv (partner_col = target_col). Partner (c,d) valid: c<d, c=tv, d=tv → tv<tv impossible → **never** |
| T2 = coeff(si,sj; d, v) | source=(si,sj), partner_col=d=tu, target_col=tv | Check: u=c, d<v (tu<tv ✓). Partner (c,d): c=u=tu, d=tu → c<d fails → **never** |
| T3 = coeff(c,d; j, v) | partner=(c,d)=(si,sj), source_col=j=tu, target_col=tv | Check: u=si, c<v (si<tv ✓). Source (si,j): si<tu=j → si<tu. But tu=si from u=si → si<si impossible → **never** |
| T4 = coeff(c,d; u, si) | partner=(c,d)=(tu,tv), target_row=u=si, source_row=si | Check: u<si (si<si? NO) → **never** |

### 3. Enumerate bracketSource

bracketSource(coeff, a,b, c,d, u,v) = coeff(a,d; u,v) if b=c, else 0.

For bracketSource = GOAL: a=si, d=sj, u=tu, v=tv. And b=c.

Source=(a,b)=(si,b), partner=(c,d)=(c,sj), need b=c and si<b=c<sj.
For adjacent sources (sj=si+1): si<b<si+1 → no integer b exists → **never**.

### 4. Conclusion

If all T1-T4 and bracketSource positions are impossible, the coefficient is
**structurally invisible** — it is not constrained by ANY half-Leibniz
equation. It can only be constrained by structural arguments:
- The spanning result (kerΦ = span{generators})
- Dimension counting (generators all have it zero)

## Python enumeration script

```python
n = 10  # example
a0, b0 = i, i+1  # GOAL source
u0, v0 = i, n     # GOAL target

# T1: coeff(a,b; u,c) when u<c and d=v
for c in range(1, n+1):
    for d in range(c+1, n+1):
        if u0 < c and d == v0 and a0 == a0 and b0 == b0:
            print(f"T1: source=({a0},{b0}), partner=({c},{d})")
            # coeff = (a0,b0; u0,c) — GOAL if c = v0
            # But c<d, so c=v0=n → d>n impossible

# ... similarly for T2, T3, T4, bracketSource
```

## Historical context

This audit was run on 2026-06-28 for GOAL = coeff(i, i+1; i, n).
Result: **structurally invisible.** The skeleton's comment at
ImageContainment.lean:2077 claiming "bracketSource via partner(i+1,n)
couples GOAL to diagonal" was found to be **incorrect** — bracketSource
gives coeff(i,n; i,n) (a diagonal), not GOAL. The correct closure path
is via the Spanning result (kerΦ span, generators all zero at this coeff).

## Generator audit (follow-up to structural invisibility)

Once structural invisibility is confirmed, the closure path is via spanning:
`kerΦ = span{generators}`, and if all generators have the coefficient zero,
then any centered D' must also have it zero.

**Before** attempting to formalize this in Lean: run a **generator audit** —
verify computationally that every generator in the spanning set has the
coefficient zero for the GOAL position.

### Method

For each generator G in {A₁,...,A_{n-1}, B, C, D, E, F}:

```lean
example : coeff_A (F:=ℚ) (n:=7) 3 3 4 3 7 = 0 := by unfold coeff_A; simp
example : coeff_B (F:=ℚ) (n:=7) 3 4 3 7 = 0 := by unfold coeff_B; simp
-- ... similarly for C, D, E, F
```

### Interpretation

- **If ALL generators have GOAL = 0 for interior i≥3**: the spanning
  argument is sound. The coefficient is genuinely zero for all centered
  derivations. Proceed to formalize via Spanning.

- **If any generator has GOAL ≠ 0 for some i**: the spanning argument
  would be insufficient. The coefficient might be a genuine degree of
  freedom in kerΦ, requiring a different proof strategy.

- **For boundary i=1,2**: target (1,n) or (2,n) ∈ I, so I_filtered does
  not constrain these coefficients. The generator audit is irrelevant here.

### Pitfall: circular dependency with Spanning

`spanning_kerPhi : kerΦ ≤ span Gset` requires `D' ∈ kerΦ`, and `kerΦ`
requires `I_filtered` — which is exactly what `centered_image_in_I` is
proving.  Do NOT attempt to `import Spanning` from within the module that
is building `I_filtered`.  The closure of structurally invisible
coefficients belongs at a HIGHER assembly level (D3/D4), not inside the
per-coefficient proof layer.
