# Mechanism Probe Before Architecture Freeze

Decide a proof's **architecture** — independent-leaf vs induction-branch,
induction axis, does-it-eat-`ih`, where do residual terms land — with a cheap
**compiled** `lean_run_code` probe, NOT on paper. This is the architecture-layer
sibling of `validate-theorem-statement.md` (which validates the theorem's
*truth*) and `structure-decomposition-paper-audit.md` (which audits a
decomposition step's *math*). Use it when the next decision is "what shape is
the module?", not "is the statement true?".

## When to load

- You must choose: is lemma X an independent leaf, or a branch inside a strong
  induction? Does X consume the main induction's `ih`?
- You need the induction axis (e.g. target width `v−u` vs source width `j−i`)
  and the precedent and the obvious choice disagree.
- A design doc (paper "blueprint") makes a *structural* claim you're about to
  freeze, and you want every such claim to be a compiled fact, not "by symmetry".

## The core move: a four-term-landing probe

For coefficient/cocycle work the half-Leibniz / bracket identity expands to a
fixed positional sum (`T1 − T2 + T3 − T4`). To learn the architecture, do NOT
attempt to close anything. Instead **inline** the project's defs (so the probe
needs no `.olean` and runs in ~1s) and prove, at CONCRETE small indices, the
*landing equalities*:

```lean
import Mathlib.Tactic
-- inline copies of bracketSource / bracketIdentity / Phi (mirror E.Core defs exactly)
def bracketSource (coeff : ℕ → ℕ → ℕ → ℕ → ℚ) (i j c d u v : ℕ) : ℚ := …
def bracketIdentity (coeff : ℕ → ℕ → ℕ → ℕ → ℚ) (i j c d u v : ℕ) : ℚ := …

-- expose what the identity reduces to at a chosen (n, source, partner, target):
example (coeff : ℕ → ℕ → ℕ → ℕ → ℚ) :
    bracketIdentity coeff 2 3 5 6 3 6 = coeff 2 3 3 5 := by unfold bracketIdentity; norm_num
example (coeff : ℕ → ℕ → ℕ → ℕ → ℚ) :
    bracketSource coeff 2 3 5 6 3 6 = 0 := by unfold bracketSource; norm_num
```

`unfold; norm_num` decides every `if`-condition at the concrete indices, leaving
exactly the surviving terms visible in the RHS. From the compiled equality you
read off, objectively:
1. which of T1..T4 vanish (condition false) and which survive;
2. the source/target of each surviving (residual) term — i.e. *where it lands*;
3. whether the residual descends (smaller width / different source) or recurses
   back onto the same object (the deadlock you're checking for).

`bracketSource = 0` (commuting partner) ⟹ `Phi = 0` ⟹ `2·bracketSource =
bracketIdentity = 0` collapses the relation; `linarith` closes the single
representative if you want to confirm "no `ih` needed" for that case.

## Read the sorry-free precedent's induction variable FIRST

Before fixing the axis, open the existing sorry-free analogue and read its
`induction'` line literally. In this project `coeffOf_nonadjacent` uses
`induction' hlen : v - u using Nat.strong_induction_on` ⟹ axis = **target
width**. Crucial distinction that bites: the **split action** happens at the
*source* midpoint, but the **induction measure** is *target* width — they are
not the same and not in conflict. `source adjacent` and `target width = 1` are
orthogonal; the real base case is usually the latter. An object that cannot
split (adjacent source = no midpoint) is the boundary delegate of the
target-width induction and does **not** eat the main `ih` — it is an independent
leaf with its own (possibly internal) acyclic structure.

## Convert "likely by symmetry" into a compiled fact before freezing

When two boundary/edge classes are related by an antiautomorphism (e.g. N_n's
`E_{i,j} ↦ E_{n+1−j, n+1−i}`), do NOT write one as compiled and its mirror as
"by symmetry" in the frozen doc. Run the cheap mirror probe. It often reveals a
*role swap* (here: the structurally-dead term flips `T4 ↔ T3`, residual landing
flips `width_stability_c ↔ width_stability_a`) that a hand-symmetry argument
would state imprecisely. Cost is one `lean_run_code` call; payoff is a blueprint
with zero non-compiled structural assertions.

## When the probe CORRECTS the design doc — ratify, don't rewrite

A probe frequently finds the blueprint's prose is not merely incomplete but
*wrong* (this session: §4.3 claimed residuals were "killed by the main-induction
`ih` among the source's own coeffs"; the probe showed residuals sit on the
*partner* source and terminate in either an interior single-identity sub-class or
a *frozen* lemma — no main `ih`). Theory-canon docs (`*-BLUEPRINT.md`,
`FINAL-THEOREM.md`) are the referee's; surface the correction explicitly,
present a patch **draft** (exact replacement text), and land it only after
ratification. Never silently rewrite a theory doc. Keep Theory (paper) and
Verification (Lean) strictly separated.

## Worked outcome (this project, D3.2 `AdjacentBase_I`)

Probes (n=6, inlined defs, `unfold; norm_num`) established, as compiled facts:
- **Interior** adjacent source: a single commuting partner makes T2/T3/T4 vanish,
  T1 = the target coeff ⟹ `coeff = 0` directly. Zero residual, zero `ih`.
- **Boundary** sources `(1,2)` and `(n−1,n)`: one term is *structurally* dead
  (`(1,2)`: T4, `u<1` impossible; `(n−1,n)`: T3, `j<v` needs `v>n`); T1/T2 is the
  extractor (T1 for `v≤n−1`, T2 for `v=n`); the sole residual is the mirror term,
  living on the **partner** source, and lands on exactly one of:
  interior-adjacent ∉I (→ the interior sub-class, single identity) or
  wide-source ∈I (→ frozen `width_stability_a` / `_c`).
- ⟹ `AdjacentBase_I` is an **independent leaf** with its own **acyclic DAG**
  (interior → boundary), depth ≤ 2, terminating in frozen Width Stability or a
  zero-residual interior identity — NOT a branch fed by the main induction `ih`,
  and NOT a 2×2 simultaneous system. Both boundary classes compiler-verified
  (the σ-mirror probe closed the last "by symmetry" gap).

The decision framing the user enforced: ask "**is X independent / does it eat
`ih`**" (architecture), not "can I close one representative" (proof) — then let
the compiler adjudicate. One representative per class, minimal-failure-rollback;
a cheap extra probe to erase any residual extrapolation before freezing.
