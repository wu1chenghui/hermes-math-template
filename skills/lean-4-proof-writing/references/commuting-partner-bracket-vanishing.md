# Commuting-Partner Bracket-Vanishing (adjacent-source `coeff = 0`)

Recipe for proving `coeffOf D i j u v = 0` for a half-derivation
`D : HalfDerivation F n` when the source `(i,j)` is adjacent/fixed and the
target `(u,v)` lies OUTSIDE the exception ideal `I = {(1,n-1),(1,n),(2,n)}`.
This is the `AdjacentBase_I` leaf of the `dim = n+5` classification (E project),
but the mechanism generalizes to any half-Leibniz coefficient identity.

## Core mechanism — commuting partner

Pick a partner source `(c,d)` that COMMUTES with `(i,j)`: `j ≠ c ∧ i ≠ d`.
Then `bracketSource = 0`, so half-Leibniz collapses to `bracketIdentity = 0`,
a pure linear relation among the coeffs of `(i,j)` and `(c,d)`.

REUSE the Core lemma `HalfDerivation.bracket_zero` (in `E.Core.HalfDerivation`,
namespace `HalfDerivation`, already opened in most files) — do NOT import the
upper-bound `Spanning` module just to get its U1 `bracketSource_commuting_eq_zero`
(that drags a heavy, wrong-direction dependency edge):

```
D.bracket_zero i j c d u v
  (1≤i)(i<j)(j≤n) (1≤c)(c<d)(d≤n) (1≤u)(u<v)(v≤n) (j≠c)(i≠d)
  : bracketIdentity D.coeff i j c d u v = 0     -- all 11 hyps via `by omega`
```

## Expansion + if-resolution

`rw [bracketIdentity_eq_expanded] at hbi` (Core, **coeff-level** signature
`(coeff)(i j c d u v)`) gives the canonical 4-term form (signs `T1 - T2 + T3 - T4`):

```
T1 = if u<c ∧ v=d then coeff i j u c else 0
T2 = if u=c ∧ d<v then coeff i j d v else 0
T3 = if u=i ∧ j<v then coeff c d j v else 0
T4 = if u<i ∧ v=j then coeff c d u i else 0
```

Resolve the four conditions with **NAMED `have c1..c4` + `rw [if_pos/if_neg]`**
(referee strict-mode; matches the project's "T1–T4 named have" discipline AND
avoids the `linter.style.whitespace` warning that inline `⟨by omega, rfl⟩`-in-`rw`
triggers):

```lean
have c1 : <cond1> := ⟨by omega, rfl⟩      -- positive, e.g. `(1:ℕ) < 3 ∧ n = n`
have c2 : ¬(<cond2>) := by omega          -- negative, e.g. `¬((1:ℕ) = 3 ∧ n < n)`
have c3 : <cond3> := ⟨rfl, by omega⟩
have c4 : ¬(<cond4>) := by omega
rw [if_pos c1, if_neg c2, if_pos c3, if_neg c4] at hbi
```

`omega` proves both negatives `¬(false-numeric ∧ _)` and linear conjunctions; use
`⟨by omega, rfl⟩` for `… ∧ n = n`-style positives (the `rfl` discharges `n = n`).
`n - 1` indices are omega-friendly given `n ≥ 5`.

## Bridge `coeffOf ↔ D.coeff`

Width lemmas and the goal speak `coeffOf`; the expansion speaks `D.coeff`. Bridge
for VALID indices via `HalfProjection.coeffOf_f D i j u v (6 validity args)`
→ `coeffOf D i j u v = D.coeff i j u v`. Use `rw [coeffOf_f …]` on the goal and
`rw [← coeffOf_f …]` to convert a `coeffOf`-stated frozen width lemma into `D.coeff`.

## Close

**`linear_combination`, NEVER `linarith`** — `F` is an arbitrary `[Field F]` with
no order, so `linarith` "fails to find a contradiction" (see `field-f-pitfalls.md`).
For `coeff_target + residual = 0` (`hbi`) and `residual = 0` (`hres`):
`linear_combination hbi - hres` closes `coeff_target = 0`. (Sign: if the relation
is `-coeff_target - residual = 0`, use `linear_combination -hbi - hres`.)

## The four edge-type templates (AdjacentBase_I), and the orientation invariant

| edge-type | residual landing | needs `hI : I_filtered`? |
|---|---|---|
| interior-`u>i` | ZERO residual (only T1 survives = the target; T2/T3/T4 vanish) | **NO** — hypothesis-free, self-closes |
| boundary-left (src `(1,2)`) | wide `∈I` residual → FROZEN `width_stability_c` (col `n`) | yes |
| boundary-right (src `(n-1,n)`) | wide `∈I` residual → FROZEN `width_stability_a` (col `n-1`); σ-mirror of left | yes |
| transfer / interior-`u<i` | Case-A relation `coeff(bdry) + coeff(interior) = 0`; grounded boundary carries the 0 | inherits boundary's |

**ORIENTATION INVARIANT (load-bearing, not stylistic):** `frozen ← boundary ← interior(u<i)`.
Ground every boundary coeff FIRST through a frozen-landing partner
(`width_stability_a/c`), THEN transfer to an interior-`u<i` coeff. NEVER pin a
boundary coeff via an interior one (or pin them mutually) — that compiles to a
degenerate `x + y = 0 / y + x = 0` 2×2 that pins nothing (compiler-verified
anti-pattern).

**interior-`u>i` is hypothesis-free** (zero residual ⇒ no `I_filtered`, no width
lemma). The `unusedVariables` linter then correctly flags `hI` — rename `_hI` to
keep a uniform leaf arity, or drop it. (Contrast the `unusedSectionVars
[CharNeTwo F]` FALSE-friend, which must NOT be silenced by removing the instance —
that breaks downstream instance synthesis.)

## Verify-before-land

Build the whole chain in `lean_run_code` first (mirror the target file's
imports / `open` / `variable` / `namespace`), THEN `patch` into the file. The
trailing `''`-column `linter.style.whitespace` warning in `lean_run_code` output
is a harness artifact on the final `end`, not from your code. Three-source verify
the landed file: LSP `lean_diagnostic_messages` (0 error / 0 sorry) + `lake build
<Module>` (`.olean` regenerated, exit 0) + `lean_verify <thm>` (axioms ⊆
`{propext, Classical.choice, Quot.sound}`, no `sorryAx`).
