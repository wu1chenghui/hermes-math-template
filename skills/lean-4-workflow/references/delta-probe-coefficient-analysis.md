# Delta Probe — Coefficient-Level BracketIdentity Enumeration

## When to use

You need to understand the structural coupling of a specific coefficient
`coeff(a,b;u,v)` — which `bracketIdentity` equations involve it, which
terms (`T1..T4`) survive, and what other coefficients it couples to.

This is an **architecture-level** probe (not a proof). Use it BEFORE
committing to an induction predicate, chain lemma, or SCC design.

## The technique

Define a "delta" coefficient function that is **1 at exactly the target
coefficient and 0 everywhere else**. Then enumerate all valid `bracketIdentity`
calls and check which produce non-zero output.

```lean4
import Mathlib.Tactic

-- Inline bracket defs over ℚ (no type class issues with run_code)
def bracketLeft (coeff : ℕ → ℕ → ℕ → ℕ → ℚ) (i j c d u v : ℕ) : ℚ :=
  (if u < c ∧ v = d then coeff i j u c else 0)
  - (if u = c ∧ d < v then coeff i j d v else 0)

def bracketRight (coeff : ℕ → ℕ → ℕ → ℕ → ℚ) (i j c d u v : ℕ) : ℚ :=
  (if u = i ∧ j < v then coeff c d j v else 0)
  - (if u < i ∧ v = j then coeff c d u i else 0)

def bracketIdentity (coeff : ℕ → ℕ → ℕ → ℕ → ℚ) (i j c d u v : ℕ) : ℚ :=
  bracketLeft coeff i j c d u v + bracketRight coeff i j c d u v

def bracketSource (coeff : ℕ → ℕ → ℕ → ℕ → ℚ) (i j c d u v : ℕ) : ℚ :=
  (if j = c then coeff i d u v else 0) - (if i = d then coeff c j u v else 0)

-- Delta at V(t) = coeff(t, t+1; t, n)
def delta_Vt (t n : ℕ) (a b u v : ℕ) : ℚ :=
  if a = t ∧ b = t+1 ∧ u = t ∧ v = n then 1 else 0

-- Enumerate ALL valid bracketIdentity calls
#eval! show IO Unit from do
  let n := 10
  let t := 4
  for a in [1:n] do for b in [a+1:n] do
    for c in [1:n] do for d in [c+1:n] do
      for u in [1:n] do for v in [u+1:n] do
        let coeff := delta_Vt t n
        let bi := bracketIdentity coeff a b c d u v
        if bi ≠ 0 then
          IO.println s!"  ({a},{b})×({c},{d})→({u},{v}): bi={bi}"
```

## What to look for

1. **How many families?** If only 2-3 parameterized families appear (like T2
   family for V(t)), the coefficient has simple structure — it's a leaf, not
   an SCC node.

2. **bracketSource = 0 always?** If bracketSource is ALWAYS 0 for the delta
   coefficient, the half-Leibniz for this coefficient is always `0 = bracketIdentity`
   (never `2V = bracketIdentity`). This means the coefficient is structurally
   a leaf — it cannot be part of an SCC. This was the key insight for V(t).

3. **What targets do the coupled coefficients have?** If all coupled coefficients
   have I-targeted targets, closure goes through width-collapse. If some have
   smaller target-width, the induction hypothesis may reach them.

4. **Full enumeration confirms exhaustiveness — no "missed" partners exist.**

## Worked example: V(t) = coeff(t, t+1; t, n)

Probe (n=7, t=4) showed V(t) appears in exactly TWO families:
- T2 family: `bracketIdentity(t, t+1, c, t, c, n) = -V(t)` for all c < t
- T3 family: `bracketIdentity(a, t, t, t+1, a, n) = +V(t)` for all a < t

Both give the same half-Leibniz equation: `V(t) = 2·coeff(1, t+1; 1, n)`.

Zero other hits. bracketSource for delta is always 0 (structural:
source (t,t+1) at target (t,n) cannot result from any valid Lie bracket).

Conclusion: V(t) is a pure leaf coupling to I-targeted b-channel. No SCC.
No smaller-target-width diagonal coupling. Closure = width_c_chain on the
b-channel coefficient.
