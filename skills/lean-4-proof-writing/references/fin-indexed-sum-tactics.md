# Fin-indexed `if`-sums and condition tactics

## Dirac-delta sum collapse: `∑ x : Fin m, if ↑p = ↑x then f x else 0 = f p`

Close with:

```lean
simp only [Fin.val_inj, Finset.sum_ite_eq, Finset.mem_univ, if_true]
```

- `Fin.val_inj` rewrites the value-level condition `↑p = ↑x` (i.e. `p.val = x.val`)
  into the element-level `p = x`, which is the shape `Finset.sum_ite_eq` expects.
- `Finset.sum_ite_eq (s) (b) (f) : (∑ a ∈ s, if b = a then f a else 0) = if b ∈ s then f b else 0`,
  then `Finset.mem_univ` + `if_true` finish.

### Do NOT use `Finset.sum_ite_eq'`

`sum_ite_eq'` expects the *mirrored* condition `if a = b` and, more importantly,
both `sum_ite_eq` and `sum_ite_eq'` need the then-branch to line up; here the
then-branch is `f x` (a function of the bound variable), so a careless
`simp [Finset.sum_ite_eq', …]` silently **does not fire** — the giveaway is the
linter reporting every simp argument as "unused". When you see "unused simp
argument" on a sum lemma, the lemma did not match — fix the lemma/condition shape,
do not just delete the arg.

Robust fallback (no name guessing): `rw [eq_comm, Finset.sum_eq_single p]` then
discharge the two side goals (`b ≠ p → if ↑p = ↑b then … else 0 = 0` via
`if_neg (fun h => hb (Fin.ext h.symm))`, and `p ∉ univ` via `absurd (mem_univ p)`).

## `Fin` proof-irrelevance in `if`-conditions and Subtype values

`(⟨k, proofA⟩ : Fin m)` and `(⟨k, proofB⟩ : Fin m)` are **definitionally equal**
(the `proof : k < m` field is a `Prop`, proof-irrelevant). Consequences:
- A goal `(big if-then-else with ⟨k,proofA⟩) = (same with ⟨k,proofB⟩)` closes by
  `rfl` / `exact` / `change` (definitional check), but a *syntactic* `rw` may refuse
  to match the differing `by omega` proof terms. Prefer the definitional tactics.
- Membership/`Subtype` witnesses like `⟨Sum.inl i, rfl⟩` close even though the two
  Subtype values carry different membership proofs.

## `change`, not `show`, when the conversion changes the goal

`show G` is meant only to *restate* the current goal for readability. When you use
it to perform a definitional reshape (e.g. unfold an evaluation map to its raw
form), the style linter flags it — use `change G` instead. Both do a defeq check;
only the lint differs.

## Nat-subtraction indices are not defeq

`(n-2)+1` is NOT definitionally `n-1` (truncated `Nat` subtraction). To rewrite an
index produced by such arithmetic, use
`rw [show (n-2)+1 = n-1 from by omega, show (n-2)+2 = n from by omega]`.
Do not reach for `congr`/`rfl` (and `congr 1` on a curried application `T a b c d`
peels the wrong argument).
