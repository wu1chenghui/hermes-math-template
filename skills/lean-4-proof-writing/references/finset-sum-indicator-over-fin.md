# Summing an indicator over `Fin` (Dirac-delta sums)

A goal shape that recurs in linear-algebra "spine" work — generator families,
`evAdjFun` coordinate proofs, `genFamily` independence, any `Basis`/`Fintype`
coefficient extraction:

    ⊢ bs p = ∑ x, if ↑p = ↑x then bs x else 0          -- p x : Fin m

## The trap

`simp [Finset.sum_ite_eq', ...]` (or `Finset.sum_ite_eq`) does NOT close this
when the THEN-branch depends on the bound variable (`bs x`, a function of `x`,
not a constant). Those lemmas expect `if a = x then c else 0` with `c` constant
in `x` (or the exact mirror form). When the then-value varies with `x`, the
rewrite never fires — the tell-tale sign is that every `simp` argument gets
flagged "unused" and the goal is left verbatim as `unsolved goals`.

Observed 2026-06-24 (Spanning.lean P3b b-channel): a prior session wrote
`simp [Finset.sum_ite_eq', Finset.mem_univ, Fin.ext_iff, eq_comm]` and it
silently left the goal open — and the project's own AGENTS.md had ALREADY
warned against exactly this lemma for exactly this sum. (See also lean-4-workflow
pitfall: status docs lag disk; verify the live error with `lean_diagnostic_messages`.)

Second trap: the condition is usually `↑p = ↑x` (`Fin.val` equality), not
`p = x`. Lemmas that key on `Fin` equality won't match `Fin.val` equality until
you bridge them (`Fin.val` is injective: `Fin.ext` / `Fin.ext_iff`).

## Fixes (any one)

1. **`Finset.sum_eq_single p` (preferred — most robust).** Gives `∑ = bs p`;
   discharge the two side goals:
   - `∀ x ∈ univ, x ≠ p → (if ↑p = ↑x then bs x else 0) = 0`:
     `intro x _ hx; rw [if_neg]; intro h; exact hx (Fin.ext h.symm)`
     (or just `simp`/`omega` once the `Fin.val` mismatch is exposed).
   - `p ∉ univ → … = 0`: `simp` (every `Fin` element is in `univ`).
   If the goal is `bs p = ∑ …` (sum on the right), take `.symm` after.

2. **`Finset.sum_congr rfl` + `Fin` uniqueness.** Normalize each term by
   `Fin.ext` + `omega`, collapsing to the single surviving index. This is the
   route the project AGENTS.md recommends for this exact sum.

3. Rewrite `↑p = ↑x` to `x = p` first (`Fin.ext_iff` + `eq_comm`), THEN
   `Finset.sum_ite_eq` — fragile, prefer (1) or (2).

## Rule of thumb

Indicator sum over a finite index whose then-branch varies with the bound
variable ⇒ reach for `Finset.sum_eq_single`, NOT `Finset.sum_ite_eq'`.
