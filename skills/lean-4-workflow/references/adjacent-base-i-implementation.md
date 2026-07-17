# AdjacentBase_I implementation recipe — commuting-partner bracketIdentity (D3.2)

This is the **implementation** layer that follows the architecture-freeze layer
(`references/mechanism-probe-before-architecture-freeze.md`). Once the D3
`AdjacentBase_I` DAG is frozen ("theory → engineering"), each per-position-class
lemma `coeffOf D i (i+1) u v = 0` (adjacent source, target `(u,v) ∉ I`) follows
ONE fixed tactic spine. Module: `E/Classification/ImageContainment.lean`.

The spine was validated by closing two representatives ("Gate 1"): a boundary
case and a transfer (interior-`u<i`) case. The remaining per-position-class
enumeration is repetition of this spine — copy a template, change the indices /
partner, keep the orientation invariant.

## The 6-step spine (per representative)

1. **Choose a commuting partner `(c,d)`** for the adjacent source `(i,i+1)`:
   require `i+1 ≠ c ∧ i ≠ d`. Pick it so the residual term lands on a wide `∈ I`
   coefficient (frozen) — see orientation invariant below.
2. **`HalfDerivation.bracket_zero`** (Core, NOT Spanning's U1 — see hygiene note):
   commuting ⟹ `bracketIdentity D.coeff i (i+1) c d u v = 0`. It takes the source
   `(i,i+1)` and partner `(c,d)`, the 9 validity hyps, then `hj_ne_c hi_ne_d`.
3. **`rw [bracketIdentity_eq_expanded] at h`** (Core, coeff-level
   `(coeff)(i j c d u v)`) → the fixed 4-term positional form:
   ```
   T1 = if u < c ∧ v = d then coeff i j u c    -- OUR source (extractor, v ≤ n-1)
   T2 = if u = c ∧ d < v then coeff i j d v    -- OUR source (extractor, v = n)
   T3 = if u = i ∧ j < v then coeff c d j v    -- PARTNER source (residual)
   T4 = if u < i ∧ v = j then coeff c d u i    -- PARTNER source (residual)
   ```
4. **Resolve the four if-conditions with NAMED `have c1..c4`**, then
   `rw [if_pos c1, if_neg c2, if_pos c3, if_neg c4] at h`. Use named haves, NOT
   inline `⟨by omega, rfl⟩`-in-`rw` (the inline form trips `linter.style.whitespace`).
   - positive conjunctions with a reflexive `=` half: `⟨by omega, rfl⟩` / `⟨rfl, by omega⟩`
   - negative conjunctions (one numeric half false): `by omega` proves the whole `¬(…∧…)`.
5. **Kill the residual** `coeff (partner) (wide target ∈ I)` with the FROZEN
   `WidthFiltration.width_stability_a` (target `1 (n-1)`) or `_c` (target `2 n`).
   Bridge `coeffOf ↔ D.coeff` with `HalfProjection.coeffOf_f` (valid indices, 6
   validity args). For a TRANSFER (interior-`u<i`) case, instead substitute an
   ALREADY-GROUNDED boundary coeff (e.g. from `boundary_rep_A`).
6. **Close with `linear_combination`, NOT `linarith`.** `F` is an arbitrary
   `[Field F]` with no order; `linarith` fails ("linarith failed to find a
   contradiction"). `linear_combination hExpansion - hResidual` closes it.

## Orientation invariant (load-bearing, blueprint §4.6)

`frozen ← boundary ← interior(u<i)`. Ground every BOUNDARY coeff FIRST via a
frozen-landing partner (`width_stability_a/c`), THEN use it as a transfer source
for an interior-`u<i` coeff. Reverse (interior pins boundary, or mutual) compiles
into a degenerate `x+y=0 / y+x=0` 2×2 that pins nothing. Sequence is not stylistic.

## Architecture-hygiene note (why bracket_zero, not Spanning.U1)

The commuting-vanishing fact also exists as `Spanning.bracketSource_commuting_eq_zero`
(U1), but `Spanning` is the heavy DOWNSTREAM upper-bound module. Importing it into
D3 just to grab one standalone lemma creates a backward dependency edge (D3 →
entire spanning proof). `HalfDerivation.bracket_zero` is the **Core** version and
delivers the stronger `bracketIdentity = 0` directly. ImageContainment then only
needs `import E.Classification.WidthFiltration` (+ Core) — acyclic, single-direction.
General rule: when you need a small standalone helper that happens to be re-stated
in a heavy downstream module, use/re-derive the UPSTREAM (Core) version.

## Signature pattern

```lean
lemma <rep> (D : HalfDerivation F n) (hn : 3 ≤ n)
    (hI : I_filtered D.coeff hn) (hn5 : 5 ≤ n) :
    coeffOf D i (i+1) u v = 0
```
`hI` is required because `width_stability_a/c` need it. `hn5 : 5 ≤ n` covers the
wide partner's width (`i+2 ≤ j`) and the final theorem's `n ≥ 5`. Keep both `hn`
and `hn5` (pass `hn hI` straight into the width lemma).

## API inventory (exact, mathlib v4.31.0-rc1 / this project)

```
HalfDerivation.bracket_zero (D)(i j c d u v)(hi hij hjn)(hc hcd hdn)(hu huv hvn)
  (hj_ne_c : j ≠ c)(hi_ne_d : i ≠ d) : bracketIdentity D.coeff i j c d u v = 0
bracketIdentity_eq_expanded (coeff)(i j c d u v) : bracketIdentity … = T1 - T2 + T3 - T4   -- Core
HalfProjection.coeffOf_f (D)(i j u v)(hi hij hjn hu huv hvn) : coeffOf D i j u v = D.coeff i j u v
WidthFiltration.width_stability_a (D)(hn:3≤n)(hI)(i j)(hi:1≤i)(hij2:i+2≤j)(hjn:j≤n) : coeffOf D i j 1 (n-1) = 0
WidthFiltration.width_stability_c (D)(hn:3≤n)(hI)(i j)(hi:1≤i)(hij2:i+2≤j)(hjn:j≤n) : coeffOf D i j 2 n = 0
```
`open MatIdx PhiOperator HalfDerivation` brings `bracket_zero`, `coeffOf`,
`coeffOf_f`, `I_filtered`, `I_target_set` unqualified; `bracketIdentity_eq_expanded`
is top-level; `width_stability_*` stay qualified (`WidthFiltration.…`).

## Comparison with internal_det_step

This recipe uses `width_stability_a/c` to kill the wide `∈ I` residual.  For the
specific target `(2,i+1)` with `i+1 < n`, there is a *self-contained* alternative
that needs no `width_stability` at all — the 3-equation 2×2 system:
`coeff(i,i+1;2,i+1) = coeff(i,n;2,n) = 2·coeff(i,n;2,n) → 0`.
See `references/internal-det-step-mechanism.md` for the full mechanism and
implementation.  Choose this pattern when:
- Target is `(2,i+1)` (the A-fires / C-fires case in `width_c_chain`)
- `i+1 < n` (guaranteed by the dispatch)
- You want zero dependency on `WidthFiltration`

## Verified templates (landed, axioms = {propext, Classical.choice, Quot.sound})

### Boundary representative — source `(1,2)`, target `(1,3) ∉ I`, partner `(3,n)`,
residual `coeff(3,n;2,n)` (wide ∈I) killed by `width_stability_c`:

```lean
lemma boundary_rep_A (D : HalfDerivation F n) (hn : 3 ≤ n)
    (hI : I_filtered D.coeff hn) (hn5 : 5 ≤ n) :
    coeffOf D 1 2 1 3 = 0 := by
  have hbi : bracketIdentity D.coeff 1 2 3 n 1 n = 0 :=
    D.bracket_zero 1 2 3 n 1 n (by omega) (by omega) (by omega) (by omega) (by omega)
      (by omega) (by omega) (by omega) (by omega) (by omega) (by omega)
  rw [bracketIdentity_eq_expanded] at hbi
  have c1 : (1 : ℕ) < 3 ∧ n = n := ⟨by omega, rfl⟩
  have c2 : ¬((1 : ℕ) = 3 ∧ n < n) := by omega
  have c3 : (1 : ℕ) = 1 ∧ 2 < n := ⟨rfl, by omega⟩
  have c4 : ¬((1 : ℕ) < 1 ∧ n = 2) := by omega
  rw [if_pos c1, if_neg c2, if_pos c3, if_neg c4] at hbi
  rw [coeffOf_f D 1 2 1 3 (by omega) (by omega) (by omega) (by omega) (by omega) (by omega)]
  have hres : D.coeff 3 n 2 n = 0 := by
    rw [← coeffOf_f D 3 n 2 n (by omega) (by omega) (by omega) (by omega) (by omega) (by omega)]
    exact WidthFiltration.width_stability_c D hn hI 3 n (by omega) (by omega) (by omega)
  linear_combination hbi - hres
```

Transfer representative — interior-`u<i` source `(3,4)`, target `(2,4) ∉ I`,
partner `(3,4)`, Case-A relation `coeff(1,2;1,3) + coeff(3,4;2,4) = 0`, boundary
zero transferred in:

```lean
lemma transfer_rep_A (D : HalfDerivation F n) (hn : 3 ≤ n)
    (hI : I_filtered D.coeff hn) (hn5 : 5 ≤ n) :
    coeffOf D 3 4 2 4 = 0 := by
  have hbz : bracketIdentity D.coeff 1 2 3 4 1 4 = 0 :=
    D.bracket_zero 1 2 3 4 1 4 (by omega) (by omega) (by omega) (by omega) (by omega)
      (by omega) (by omega) (by omega) (by omega) (by omega) (by omega)
  rw [bracketIdentity_eq_expanded] at hbz
  have c1 : (1 : ℕ) < 3 ∧ (4 : ℕ) = 4 := ⟨by omega, rfl⟩
  have c2 : ¬((1 : ℕ) = 3 ∧ (4 : ℕ) < 4) := by omega
  have c3 : (1 : ℕ) = 1 ∧ (2 : ℕ) < 4 := ⟨rfl, by omega⟩
  have c4 : ¬((1 : ℕ) < 1 ∧ (4 : ℕ) = 2) := by omega
  rw [if_pos c1, if_neg c2, if_pos c3, if_neg c4] at hbz
  rw [coeffOf_f D 3 4 2 4 (by omega) (by omega) (by omega) (by omega) (by omega) (by omega)]
  have hbnd : D.coeff 1 2 1 3 = 0 := by
    rw [← coeffOf_f D 1 2 1 3 (by omega) (by omega) (by omega) (by omega) (by omega) (by omega)]
    exact boundary_rep_A D hn hI hn5
  linear_combination hbz - hbnd
```

## Workflow that yielded zero-rework landing

`#check`/outline API recon → assemble the full chain in `lean_run_code` first
(self-contained, imports resolve to existing `.olean`) → only after `success:true`
`patch` into the file → three-source verify (LSP `lean_diagnostic_messages` 0
error/0 sorry · `lake build E.Classification.ImageContainment` exit 0 · `lean_verify`
each lemma axioms-clean). The only iteration needed was `linarith → linear_combination`.
